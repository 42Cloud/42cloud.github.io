---
title: Open vSwitch 源码阅读之 handle_flow_mod
categories: 
    - SDN
tags:
    - OVS
---



handle_flow_mod 函数的调用关系

<!--more-->

## 数据结构

```c

enum ofperr  // OpenFlow error codes
enum ofpraw  // Raw identifiers for OpenFlow messages
struct ofconn  // An OpenFlow connection
struct ofp_header  // Header on all OpenFlow packets
struct ofproto  // An OpenFlow switch
struct ofputil_flow_mod  // Protocol-independent flow_mod
struct ofpbuf  // Buffer for holding arbitrary data
struct openflow_mod_requester  // The source of an OpenFlow request
struct vl_mff_map  // Variable length mf_fields mapping map

```
	
## 函数调用层次关系
```c

handle_flow_mod()
    ofconn_get_ofproto()
    reject_slave_controller()
    /* Initializes ofpbuf */
    ofpbuf_use_stub()
	ofpbuf_use__()
    ofputil_decode_flow_mod()
        ofpbuf_const_initializer()
		ofpraw_pull_assert()
        
        if raw == OFPRAW_OFPT11_FLOW_MOD
            ...
        if raw == OFPRAW_OFPT10_FLOW_MOD
            /* Get the ofp10_flow_mod. */
            ofpbuf_pull() 
            /* Translate the rule. */
            ofputil_match_from_ofp10_match()
            ofputil_normalize_match()        
        if raw == OFPRAW_NXT_FLOW_MOD
            ...
            
        ofpacts_pull_openflow_instructions()
			if version == OFP10_VERSION
				ofpacts_pull_openflow_actions__()
	            	ofpbuf_try_pull()
	            	ofpacts_decode()
	            	ofpacts_verify()
			if version == OFP11_VERSION
				...
        ofputil_decode_flow_mod_flags()
        ofpacts_check_consistency()
    handle_flow_mod__()
        ofproto_flow_mod_init()
    ofpbuf_uninit()
	
```
## 其他处理函数以及相关的数据结构

### handle_features_request()
```c

   struct ofputil_switch_features   /* Abstract ofp_switch_features. */
   struct ofputil_table_features  /* Abstract ofp_table_features. */
   
```

### handle_get_config_request()
```c

   struct ofputil_switch_config  /* Abstract struct ofp_switch_config. */

```

### handle_packet_out()
```c

   struct ofputil_packet_out  /* Abstract packet-out message. */
   
```

注：只关注 openflow1.0 协议的相关内容，为一些相关数据结构以及函数添加注释作为备忘（略去一些较为简单的处理函数）

