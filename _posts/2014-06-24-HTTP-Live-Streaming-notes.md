---
layout: post
title: HTTP Live Streaming 笔记
tagline: ""
category: null
tags: []
published: true
---
# Playlist

* m3u8：扩展的M3U文件，以行分割，每一行结尾是LF或者CR LF。每一行表示一个URI，空行（被忽略）或者以#开头（注释或者tag/标签）。不能包含空格（？）。
* URI：表示一个分片或者一个playlist文件。可能是相对路径，若是，则根据该playlist路径拼凑完整路径。
* Media Playlist：仅包含分片的palylist
* Master Playlist：仅包含playlist的playlist
* Tags：以#EXT开头，所有其他的以#开头的行均是注释。
* duartion：所有分片duration之和
* 编码：HTTP Content-Type "application/vnd.apple.mpegurl"的m3u8文件为UTF-8编码，HTTP Content-Type "audio/mpegurl"的m3u文件为US-ASCII编码

* AttributeName=AttributeValue,AttributeName=AttributeValue,…

	AttributeName不能包含空格，只能由A～Z和"-"组成，同一个AttributeName在一个attribute list中不能重复。
	AttributeValue只能是10进制数字，十六进制数字(0x/0X)，十进制浮点数，引号包裹的字符串（“），没有引号包裹的可枚举字符串（？），十进制分辨率(widthxheight)

## EXTM3U
区分M3U8和M8U文件的标签

## EXTINF
标识分片的duration，格式：

    #EXTINF:<duration>,<value>

duration是十进制数字或者十进制浮点数，单位为秒，协议版本号低于3的仅使用十进制证书
value可选

## EXT-X-BYTERANGE:\<n>[@o]
表示sub-range of resource
n表示sub-range长度，单位为字节
o是一个十进制整数，标识sub-range的起始位置，从resource的开始位置的偏移量，单位为字节
协议版本号从4开始使用该标签

## EXT-X-TARGETDURATION:\<s>
表示playlist文件中可能出现的最大duration
s是最大duration，以秒为单位

## EXT-X-MEDIA-SEQUENCE:\<number>
标识playlist文件中出现的第一个sequence number，仅出现一次

## EXT-X-KEY:<attribute-list>
作用域到下一个包含相同KEYFORMAT attribute的EXT-X-KEY标签之前
METHOD：NONE, AES-128, SAMPLE-AES，若为NONE，则必须包含URI,VI,KEYFORMAT,KEYFORMATVERSIONS；若为AES-128，则必须包含URI，可选VI；若为SAMPLE-AES，可选VI
URI：key的URI
IV：Initialization Vector，十六进制整数，协议从版本号2开始包含此变量
KEYFORMAT：引用字符串
KEYFORMATVERSIONS：引用字符串，出现在协议版本号5中

## EXT-X-PROGRAM-DATE-TIME:<YYYY-MM-DDThh:mm:ssZ>
接下来的第一个的分片时间

## EXT-X-ALLOW-CACHE:<YES|NO>
客户端是否可以缓存分片用于回放，仅能出现一次

## EXT-X-PLAYLIST-TYPE:<EVENT|VOD>
playlist是否可修改

## EXT-X-ENDLIST
表示不再有分片，仅能出现一次

## EXT-X-MEDIA:<attribute-list>
用于表示同一内容的不同选择(alternative reditions)，比如只包含音频的英语，法语和西班牙语音轨，或者仅包含视频的两个摄像机机位，format相同
URI可选
TYPE三选一，VIDEO/AUDIO/SUBTITLES，同一GROUP的必须相同
GROUP-ID引用字符串
LANGUAGE可选 引用字符串，参考RFC 5646，若AUTOSELECT为YES，则LANGUAGE必须有效
NAME引用字符串，同一GROUP的必须不同
DEFAULT二选一YES/NO，同一GROUP仅有一个为YES
AUTOSELECT二选一YES/NO
FORCED二选一YES/NO 可选
CHARACTERISTICS引用字符串 可选

## EXT-X-STREAM-INF:<attribute-list>\<URI>
表示URI为playlist
BANDWIDTH必须包含，十进制整数，单位为比特
PROGRAM-ID Master Playlist中通过不同的PROGRAM-ID表示相同presentation的不同编码
CODECS引用字符串，value通过逗号分割，建议包含
RESOLUTION 视频分辨率
AUDIO 引用字符串，和GROUP-ID相同
VIDEO 引用字符串，和GROUP-ID相同
SUBTITLES 引用字符串，和GROUP-ID相同

## EXT-X-DISCONTUNUITY
媒体信息发生变化，媒体信息包括文件格式，tracks的数量和类型，编码参数，编码顺序(encoding sequence)，时间戳顺序(timestamp sequence)

## EXT-X-I-FRAMES-ONLY
表示分片只包含I帧，协议从版本号4开始包含此标签

## EXT-X-MAP:<attribute-list>
标识如何获取分片的PAT/PMT，对该标签之后的分片有效
仅和EXT-X-I-FRAMES-ONLY一起出现，建议只用在没有PAT/PMT开头的分片
协议从版本号5开始使用该标签
URI引用字符串
BYTERANGE引用字符串

## EXT-X-I-FRAME-STREAM-INF:<attribute-list>
表示I帧列表
URI
BANDWIDTH

## EXT-X-VERSION:<n>
协议版本号

# 额外的注意：

1. media segment：分片
2. 编码 MPEG-2 Transport Stream/MPEG audio elementary stream/WebVTT（字幕）
3. TS分片必须仅包含一个MPEG-2 Program。每一个TS分片都应当以PAT和PMT开头。视频分片必须包含一个I帧（完整GOP？）
5. TS或者audio elementacontentry stream分片必须保证sequence number连续，时间戳连续，除非是第一个分片或者出现了EXT-X-DISCONTINUITY标签。
