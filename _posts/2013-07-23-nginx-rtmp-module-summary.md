---
layout: post
title: "nginx-rtmp-module summary"
description: ""
category: 
tags: []
---
{% include JB/setup %}

##项目主页:
[https://github.com/arut/nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module)

##项目介绍：
nginx-rtmp-module是为nginx开发的一组模块，实现基于nginx的流媒体服务器，包括流发布，转发，录制等功能。具体参考项目主页及项目wiki。

注意，该项目正在开发中，某些功能尚不稳定，代码结构也可能在将来发生较大的变化。

##总结
###配置解析
配置解析要从rtmp命令开始，这是一个顶级命令，和http、events等平级。通过阅读代码能看出来，该命令的解析参考了http的解析流程，具体代码位于nginx_rtmp.c中，如果以前对http命令解析过程有了解，相信这里也能很快理解。

nginx-rtmp-module项目实现的是一组模块，这里为了描述方便，仅用短命名表示各模块，比如core模块，hls模块等，需要注意的是nginx_rtmp.c中声明的模块，这里称呼他为根模块。

这是一份常见也很典型的rtmp模块配置：
	
	rtmp {
	    server {

	        listen 1935; # 监听1935端口
	
	        application live {
	            live on; # 启用直播   
	
	            hls on;  # 启用HLS
	            hls_path /tmp/tv26; # HLS缓存路径
	            
	            recorder record_tv26{ # 启用录制
	                record all; # 收录音视频
	                record_path /tmp/rec; # 收录路径
	            }   
	        }   
	    }   
    
具体解析部分如下：
    
	/* rtmp{}块解析函数，一切与rtmp有关的配置解析从这里开始 */
	static char *
	ngx_rtmp_block(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
	{
	    char                        *rv;
	    ngx_uint_t                   i, m, mi, s;
	    ngx_conf_t                   pcf;
	    ngx_array_t                  ports;
	    ngx_rtmp_listen_t           *listen;
	    ngx_rtmp_module_t           *module;
	    ngx_rtmp_conf_ctx_t         *ctx;
	    ngx_rtmp_core_srv_conf_t    *cscf, **cscfp;
	    ngx_rtmp_core_main_conf_t   *cmcf;
	
		/* 
		typedef struct {
		    void                  **main_conf;
		    void                  **srv_conf;
		    void                  **app_conf;
		} ngx_rtmp_conf_ctx_t;
		 * 这个结构体再配置解析过程中见到非常多，server命令和application命令解析时也会创建一个这样的结构体，需要注意的时，这个结构体中main_conf是一次rtmp{}块解析过程中唯一的，其他命令解析到的配置将直接修改该main_conf，与另两个结构体不同。
		 */
	    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_rtmp_conf_ctx_t));
	    if (ctx == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	    *(ngx_rtmp_conf_ctx_t **) conf = ctx;
	
	    /* count the number of the rtmp modules and set up their indices */
	    /* 计算RTMP模块总数，并为模块编号 */
	
	    ngx_rtmp_max_module = 0;
	    for (m = 0; ngx_modules[m]; m++) {
	        if (ngx_modules[m]->type != NGX_RTMP_MODULE) {
	            continue;
	        }
	
	        ngx_modules[m]->ctx_index = ngx_rtmp_max_module++;
	    }
	
	
	    /* the rtmp main_conf context, it is the same in the all rtmp contexts */
	    /* 再次强调，创建main_conf上下文变量，全局唯一 */
	    ctx->main_conf = ngx_pcalloc(cf->pool,
	                                 sizeof(void *) * ngx_rtmp_max_module);// 这里使用了max创建数组
	    if (ctx->main_conf == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	
	    /*
	     * the rtmp null srv_conf context, it is used to merge
	     * the server{}s' srv_conf's
	     * 创建srv_conf上下文变量
	     */
	
	    ctx->srv_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_rtmp_max_module);
	    if (ctx->srv_conf == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	
	    /*
	     * the rtmp null app_conf context, it is used to merge
	     * the server{}s' app_conf's
	     * 创建app_conf上下文变量
	     */
	
	    ctx->app_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_rtmp_max_module);
	    if (ctx->app_conf == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	
	    /*
	     * create the main_conf's, the null srv_conf's, and the null app_conf's
	     * of the all rtmp modules
	     */
	    /* 遍历所有的RTMP模块，保证所有模块对应的conf不为空 */
	
	    for (m = 0; ngx_modules[m]; m++) {
	        if (ngx_modules[m]->type != NGX_RTMP_MODULE) {
	            continue;
	        }
	
	        module = ngx_modules[m]->ctx;
	        mi = ngx_modules[m]->ctx_index;
	
	        // 分别调用各模块的:
	        // create_main_conf，
	        // create_srv_conf
	        // create_app_conf
	        if (module->create_main_conf) {
	            ctx->main_conf[mi] = module->create_main_conf(cf);
	            if (ctx->main_conf[mi] == NULL) {
	                return NGX_CONF_ERROR;
	            }
	        }
	
	        if (module->create_srv_conf) {
	            ctx->srv_conf[mi] = module->create_srv_conf(cf);
	            if (ctx->srv_conf[mi] == NULL) {
	                return NGX_CONF_ERROR;
	            }
	        }
	
	        if (module->create_app_conf) {
	            ctx->app_conf[mi] = module->create_app_conf(cf);
	            if (ctx->app_conf[mi] == NULL) {
	                return NGX_CONF_ERROR;
	            }
	        }
	    }
	
	    // 备份ngx_conf_t，在rtmp{}块解析完成后恢复
	    pcf = *cf;
	    cf->ctx = ctx;
	
	    // 调用各RTMP模块的preconfiguration
	    for (m = 0; ngx_modules[m]; m++) {
	        if (ngx_modules[m]->type != NGX_RTMP_MODULE) {
	            continue;
	        }
	
	        module = ngx_modules[m]->ctx;
	
	        if (module->preconfiguration) {
	            if (module->preconfiguration(cf) != NGX_OK) {
	                return NGX_CONF_ERROR;
	            }
	        }
	    }
	
	    /* parse inside the rtmp{} block */
	    /* 进入rtmp{}块中继续解析配置，仅解析cmd_type=NGX_RTMP_MAIN_CONF的指令，比如server{}块。
	     * 这里要回顾一下ngx_rtmp_conf_ctx_t结构体，因为接下来的解析过程，实际上就是填充这三个结构体的过程。
	     * 结构体包括三个数组，分别是main_conf，srv_conf和app_conf，
	     * main_conf实际上一个ngx_rtmp_*_main_conf_t的数组，其中最重要的可能就是core模块的结构，
	      
			typedef struct {
				ngx_array_t             servers;    // ngx_rtmp_core_srv_conf_t 
				ngx_array_t             listen;     // ngx_rtmp_listen_t 
				
				ngx_array_t             events[NGX_RTMP_MAX_EVENT];
				
				ngx_hash_t              amf_hash;
				ngx_array_t             amf_arrays;
				ngx_array_t             amf;
			} ngx_rtmp_core_main_conf_t;
	     
	     * 解析到的server{}块配置会保存到core模块的servers数组中
	     * 同样的，srv_conf是一个ngx_rtmp_*_srv_main_t的数组，其中最重要的是core模块中针对server配置的结构体，
	     
			typedef struct ngx_rtmp_core_srv_conf_s {
				ngx_array_t             applications; // ngx_rtmp_core_app_conf_t
				...			
				ngx_rtmp_conf_ctx_t    *ctx;
			} ngx_rtmp_core_srv_conf_t;
	     
	     * 解析到的application{}块会保存到相应server块的applications数组中 */
	
	    cf->module_type = NGX_RTMP_MODULE;
	    cf->cmd_type = NGX_RTMP_MAIN_CONF;
	    rv = ngx_conf_parse(cf, NULL);
	
	    if (rv != NGX_CONF_OK) {
	        *cf = pcf;
	        return rv;
	    }
	
	
	    /* init rtmp{} main_conf's, merge the server{}s' srv_conf's */
	
	    // cmcf => core module conf
	    cmcf = ctx->main_conf[ngx_rtmp_core_module.ctx_index];
	    cscfp = cmcf->servers.elts;
	
	    /* 合并各RTMP模块配置 
	     * 这里的合并有一点需要注意的是，core模块的配置必须首先被合并，这个通过写config文件来实现，只有这样，才能保证上一轮解析中的配置全部合并到ctx的三大数组中。
	     * 跟模块不在这一轮合并中出现，因为根模块的类型不是NGX_RTMP_MODULE而是NGX_CORE_MODULE
	     */
	    for (m = 0; ngx_modules[m]; m++) {
	        if (ngx_modules[m]->type != NGX_RTMP_MODULE) {
	            continue;
	        }
	
	        module = ngx_modules[m]->ctx;
	        // mi => module_index
	        mi = ngx_modules[m]->ctx_index;
	
	        /* init rtmp{} main_conf's */
	        /* 调用各main模块的init_main_conf */
	
	        cf->ctx = ctx;
	
	        // 调用各RTMP main模块的init_main_conf，因为全局统一使用一个main_conf数组，因此这里不需要“merge”
	        if (module->init_main_conf) {
	            rv = module->init_main_conf(cf, ctx->main_conf[mi]);
	            if (rv != NGX_CONF_OK) {
	                *cf = pcf;
	                return rv;
	            }
	        }
	
	        /* 分别调用各模块merge_srv_conf和merge_app_conf，如果需要的话
	         * ngx_parse_conf产生的servers用在这里*/
	        for (s = 0; s < cmcf->servers.nelts; s++) {
	
	            /* merge the server{}s' srv_conf's */
	
	            cf->ctx = cscfp[s]->ctx;
	
	            if (module->merge_srv_conf) {
	                rv = module->merge_srv_conf(cf,
	                                            ctx->srv_conf[mi],
	                                            cscfp[s]->ctx->srv_conf[mi]);
	                if (rv != NGX_CONF_OK) {
	                    *cf = pcf;
	                    return rv;
	                }
	            }
	
	            if (module->merge_app_conf) {
	
	                /* merge the server{}'s app_conf */
	
	                /*ctx->app_conf = cscfp[s]->ctx->loc_conf;*/
	
	                rv = module->merge_app_conf(cf, 
	                                            ctx->app_conf[mi],
	                                            cscfp[s]->ctx->app_conf[mi]);
	                if (rv != NGX_CONF_OK) {
	                    *cf = pcf;
	                    return rv;
	                }
	
	                /* merge the applications{}' app_conf's 
	                 * 解析applications数组
	                 * cscfp = core server conf pointer*/
	
	                cscf = cscfp[s]->ctx->srv_conf[ngx_rtmp_core_module.ctx_index];
	
	                /* 递归的解析applications数组，因为application有可能有子块，比如recorder{}块 */
	                rv = ngx_rtmp_merge_applications(cf, &cscf->applications,
	                                            cscfp[s]->ctx->app_conf,
	                                            module, mi);
	                if (rv != NGX_CONF_OK) {
	                    *cf = pcf;
	                    return rv;
	                }
	            }
	
	        }
	    }
	
	
	    /* 初始化events数组和amf数组 */
	    if (ngx_rtmp_init_events(cf, cmcf) != NGX_OK) {
	        return NGX_CONF_ERROR;
	    }
	
	    /* 调用各RTMP模块的postconfiguration */
	    for (m = 0; ngx_modules[m]; m++) {
	        if (ngx_modules[m]->type != NGX_RTMP_MODULE) {
	            continue;
	        }
	
	        module = ngx_modules[m]->ctx;
	
	        if (module->postconfiguration) {
	            if (module->postconfiguration(cf) != NGX_OK) {
	                return NGX_CONF_ERROR;
	            }
	        }
	    }
	
	    // 恢复conf
	    *cf = pcf;
	
	    /* 初始化RTMP各事件响应函数，AMF各事件响应函数及hash表 */
	    if (ngx_rtmp_init_event_handlers(cf, cmcf) != NGX_OK) {
	        return NGX_CONF_ERROR;
	    }
	
	    /* 初始化端口数组 */
	    if (ngx_array_init(&ports, cf->temp_pool, 4, sizeof(ngx_rtmp_conf_port_t))
	        != NGX_OK)
	    {
	        return NGX_CONF_ERROR;
	    }
	
	    listen = cmcf->listen.elts;
	
	    // 添加listen地址到port数组
	    for (i = 0; i < cmcf->listen.nelts; i++) {
	        if (ngx_rtmp_add_ports(cf, &ports, &listen[i]) != NGX_OK) {
	            return NGX_CONF_ERROR;
	        }
	    }
	
	    /* 将port中的监听地址添加到nginx的监听列表 */
	    return ngx_rtmp_optimize_servers(cf, &ports);
	}
仅看根模块的rtmp{}块解析还无法全面了解配置解析过程，下面贴出另一个重要模块，core模块，的server{}块和applicaiton{}块解析过程，他们在ngx_rtmp_core_module.c中。

	static char *
	ngx_rtmp_core_server(ngx_conf_t *cf, ngx_command_t *cmd, void *conf/*srv_conf*/)
	{
	    char                       *rv;
	    void                       *mconf;
	    ngx_uint_t                  m;
	    ngx_conf_t                  pcf;
	    ngx_rtmp_module_t          *module;
	    ngx_rtmp_conf_ctx_t        *ctx, *rtmp_ctx;
	    ngx_rtmp_core_srv_conf_t   *cscf, **cscfp;
	    ngx_rtmp_core_main_conf_t  *cmcf;
	
	    // 每解析到一个server就创建一个新的conf_ctx
	    ctx = ngx_pcalloc(cf->pool, sizeof(ngx_rtmp_conf_ctx_t));
	    if (ctx == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	    rtmp_ctx = cf->ctx;
	    // 使用rtmp级conf_ctx的main_conf覆盖新建的本server的main_conf，以保证唯一的main_conf数组
	    ctx->main_conf = rtmp_ctx->main_conf;
	
	    /* the server{}'s srv_conf */
	    /* 为srv_conf和app_conf申请内存 */
	
	    ctx->srv_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_rtmp_max_module);
	    if (ctx->srv_conf == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	    ctx->app_conf = ngx_pcalloc(cf->pool, sizeof(void *) * ngx_rtmp_max_module);
	    if (ctx->app_conf == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	    // 初始化srv_conf和app_conf
	    // 调用各RTMP模块的create_srv_conf和create_app_conf
	    for (m = 0; ngx_modules[m]; m++) {
	        if (ngx_modules[m]->type != NGX_RTMP_MODULE) {
	            continue;
	        }
	
	        module = ngx_modules[m]->ctx;
	
	        if (module->create_srv_conf) {
	            mconf = module->create_srv_conf(cf);
	            if (mconf == NULL) {
	                return NGX_CONF_ERROR;
	            }
	
	            ctx->srv_conf[ngx_modules[m]->ctx_index] = mconf;
	        }
	
	        if (module->create_app_conf) {
	            mconf = module->create_app_conf(cf);
	            if (mconf == NULL) {
	                return NGX_CONF_ERROR;
	            }
	
	            ctx->app_conf[ngx_modules[m]->ctx_index] = mconf;
	        }
	    }
	
	    /* the server configuration context */
	    /* 将新申请的conf_ctx放入main_conf的servers数组
	     * servers数组是ngx_rtmp_core_srv_conf_t的集合*/
	    cscf = ctx->srv_conf[ngx_rtmp_core_module.ctx_index];
	    cscf->ctx = ctx; // 自我链接，用于下一步放入servers
	    /* 至此，解析server{}块配置的过程实际上就结束了，需要合并配置时，main_conf[core_module]->servers[...]->ctx->srv_conf[core_module]即可找到每一server{}块的配置，
	     * application{}块的解析也同样如此，不同的时application块中也有applications[]数组，因此需要递归的合并子模块的配置
	     */
	
	    cmcf = ctx->main_conf[ngx_rtmp_core_module.ctx_index];
	
	    cscfp = ngx_array_push(&cmcf->servers);/* 由于main_conf是直接覆盖而非另外生成，因此这里放入的servers数组就是输入的main_conf */
	    if (cscfp == NULL) {
	        return NGX_CONF_ERROR;
	    }
	
	    *cscfp = cscf;
	
	    /* parse inside server{} 
	     * 进入server{}块内部继续解析，这里仅解析NGX_RTMP_SRV_CONF类型的命令，包括listen和application等命令。
	    */
	
	    pcf = *cf;
	    cf->ctx = ctx;
	    cf->cmd_type = NGX_RTMP_SRV_CONF;
	
	    rv = ngx_conf_parse(cf, NULL);
	
	    *cf = pcf;
	
	    return rv;
	}

### TODO 接受PUSH RTMP流：

##参考：
Nginx模块开发入门 [http://blog.codinglabs.org/articles/intro-of-nginx-module-development.html](http://blog.codinglabs.org/articles/intro-of-nginx-module-development.html)

Emiller's Guide To Nginx Module Development [http://www.evanmiller.org/nginx-modules-guide.html](http://www.evanmiller.org/nginx-modules-guide.html)
