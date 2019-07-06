---
title: OVS 源码阅读之 handle_flow_mod__
categories:
    - SDN
tags:
    - OVS
---






handle_flow_mod__ 函数作为 handle_flow_mod 函数中的重要部分，需要进一步的分析
<!--more-->

## 数据结构

```c

struct ofproto_flow_mod  /* flow_mod with execution context. Allocated by 'init' phase, may be freed after 'start' phase, as these are not needed for 'revert' nor 'finish'. */
struct ofputil_flow_mod  /* Protocol-independent flow_mod. */
struct oftable  /* A flow table within a "struct ofproto". */
struct cls_rule  /* A rule to be inserted to the classifier. */
enum oftable_flags  /* OpenFlow table flags: Hidden , Read-only */
struct ofputil_flow_mod  /* Protocol-independent flow_mod. */
struct rule  /* Where this rule resides in an OpenFlow switch. */
enum ofputil_flow_mod_flags  /* Protocol-independent flow_mod flags. */
struct cls_conjunction
struct ofpact
struct ofp10_flow_mod   /* Flow setup and teardown (controller -> datapath). */
    ovs_be32 buffer_id           /* Buffered packet to apply to (or -1). Not meaningful for OFPFC_DELETE*. */
uint64_t cookie; /* Opaque controller-issued identifier. */
OFPERR_OFPBRC_BUFFER_UNKNOWN  /* OF1.0+(1,8).  Specified buffer does not exist. */
OFPERR_OFPBRC_EPERM      /* OF1.0+(1,5).  Permissions error. */

```

## 函数调用关系

### 大致的函数调用

```c

handle_flow_mod__()
    ofproto_flow_mod_init()
    ofproto_flow_mod_start()
    ofproto_bump_tables_version()
    ofproto_flow_mod_finish()
    ofmonitor_flush()

```

### ofproto_flow_mod_init

```c
    ofproto_flow_mod_init()
    {
        case OFPFC_ADD
        {
            /* add_flow_init(), add_flow_start(), add_flow_revert(), and add_flow_finish()
             * implement OFPFC_ADD and the cases for OFPFC_MODIFY and OFPFC_MODIFY_STRICT
             * in which no matching flow already exists in the flow table.
             */
            add_flow_init()
                /* Chooses an appropriate table for 'match' within 'ofproto'. */
                rule_choose_table() 
                cls_rule_init()
                    cls_rule_init__()
                        rculist_init()
                        ovsrcu_init()
                    minimatch_init()
                /* Create a new rule. */
                ofproto_rule_create()
                    cls_rule_move()
                    ovs_refcount_init()
                    ovs_list_init()
                    /* meta flow */
                    mf_vl_mff_ref()
                get_conjunctions() /* ?  */
                return 0
        }
        case OFPFC_MODIFY
        {   
            modify_flows_init_loose()
                rule_criteria_init()
                    /* cls_rule. */
                    cls_rule_init()
                        rculist_init()
                        ovsrcu_init()
                rule_criteria_require_rw()
                add_flow_init()
                    ...
        }
        case OFPFC_MODIFY_STRICT
        {
            modify_flow_init_strict()
                rule_criteria_init()
                rule_criteria_require_rw()
                add_flow_init()
        }
        case OFPFC_DELETE
        {
            delete_flows_init_loose()
                rule_criteria_init()
                    /* cls_rule. */
                    cls_rule_init()
                        rculist_init()
                        ovsrcu_init()
                rule_criteria_require_rw()
        }
        case OFPFC_DELETE_STRICT
        {
            delete_flows_init_strict()
                rule_criteria_init()
                rule_criteria_require_rw()
        }
        default:
            ...
            
        /* OFPFC_ADD: fm->buffer_id = -1 */
        if(!error && check_buffer_id && fm->buffer_id != UINT32_MAX)
        {
            error = OFPERR_OFPBRC_BUFFER_UNKNOWN
        }
        if error   /* OFPFC_ADD: error = 0 */
            ofproto_flow_mod_uninit()
                ofproto_rule_unref()
                rule_criteria_destroy()
                free()
        return error
    }
```

### ofproto_flow_mod_start

```c

    ofproto_flow_mod_start()
    {
        /* set ofm->old_rules and ofm->new_rules */
        rule_collection_init()
        case OFPFC_ADD
        {
            add_flow_start()
                rule_get_actions()
                /* check actions */
                ofproto_check_ofpacts()
                /* Check for the existence of an identical rule. */
                rule_from_cls_rule()
                if !old_rule
                    /* without old rule */
                else
                    ofm->modify_cookie = true
                rule_collection_add()
                /* evicted rule or replaced rule */
                replace_rule_start()
                return 0
        }
        case OFPFC_MODIFY
        {
            modify_flows_start_loose()
                collect_rules_loose()
                if !error
                    error = modify_flows_start__()
                    {
                        rule_get_actions()
                        ofproto_check_ofpacts()
                        ...     
                    }
                if error
                    rule_collection_destroy
                return error  /* error = 0 */
        }
        case OFPFC_MODIFY_STRICT
        {
            modify_flow_start_strict()
                collect_rules_strict()
                if !error
                    error = modify_flows_start__()
                    {
                        ...
                    }
                return error  /* error = 0 */
        }
        case OFPFC_DELETE
        {
            delete_flows_start_loose()
                collect_rules_loose()
                    rule_collection_init()
                delete_flows_start__()
                    cls_rule_make_invisible_in_version()
                    ofproto_rule_remove__()
                        cookies_remove()
                        eviction_group_remove_rule()
                        ovs_list_remove()
                        ofproto_group_lookup()
                        group_remove_rule()
        }
        case OFPFC_DELETE_STRICT
        {
            delete_flow_start_strict()
                collect_rules_strict()
                delete_flows_start__()
        }
        default:
            ...
        /* Release resources not needed after start. */
        ofproto_flow_mod_uninit()
        {
            ofproto_rule_unref()
            {
                ovs_assert()
                ovsrcu_postpone()
                    rule_destroy_cb()
                        /* Send rule removed if needed. */
                        ofproto_rule_send_removed()
                        {
                            minimatch_expand()
                            calc_duration()
                            rule_get_stats()
                            connmgr_send_flow_removed()
                            {
                                if ofconn_receives_async_msg()
                                    ofputil_encode_flow_removed()
                                    ofconn_send_reply()
                                        ofconn_send()
                                            ofpmsg_update_length()
                                            rconn_send()
                            }
                        }
                        mf_vl_mff_unref()
                        ofproto_rule_destroy__()
            }
            rule_criteria_destroy()
                cls_rule_destroy
        }
        if error    /* OFPFC_ADD: error = 0 */
            rule_collection_destroy()
            rule_collection_destroy()
        return error
        
    }

```

### ofproto_flow_mod_finish

```c
    ofproto_flow_mod_finish()
    {
        case OFPFC_ADD
        {
            add_flow_finish()
                replace_rule_finish()
                {
                    /* OFPFC_ADD:replaced_role = NULL */
                    replaced_rule = (old_rule && old_rule->removed_reason != OFPRR_EVICTION)
                    ? old_rule : NULL;
                    /* ? */
                    rule_insert()
                    learned_cookies_inc()
                    if old_rule
                        learned_cookies_dec()
                            if replaced_rule
                                if event != NXFME_MODIFIED || changed_actions || changed_cookie
                                    ofmonitor_report()
                            else
                                /* XXX: This is slight duplication with delete_flows_finish__() */
                                ofmonitor_report()
                                    ....
                }
                learned_cookies_flush()
        }
        case OFPFC_MODIFY
        case OFPFC_MODIFY_STRICT
            modify_flows_finish()
        case OFPFC_DELETE
        case OFPFC_DELETE_STRICT
        {
            delete_flows_finish()
                delete_flows_finish__()
                {
                    if rule_collection_n(rules)
                    {
                        ofmonitor_report()
                            update = NXFMF_DELETE
                        send_table_status()
                        learned_cookies_dec()
                            learned_cookies_update__()
                        remove_rules_postponed()
                        {
                            ...
                        }
                        learned_cookies_flush()
                        {
                            match_init_catchall()
                            rule_criteria_init()
                            rule_criteria_require_rw()
                            collect_rules_loose()
                            rule_criteria_destroy()
                            delete_flows__() /* rule_collection_n(rules) = 0 */  
                                if rule_collection_n(rules)
                                {
                                    delete_flows_start__()
                                    ofproto_bump_tables_version()
                                    delete_flows_finish__()
                                    ofmonitor_flush()
                                }
                        }
                    }
                }
        }
        rule_collection_destroy()
        if req
            ofconn_report_flow_mod()
    }
```

## 总结

以上就是 handle_flow_mod__ 函数的调用关系，具体细节还需要深入了解
