bpftool feature
```
Scanning system configuration...
bpf() syscall restricted to privileged users
JIT compiler is enabled
JIT compiler hardening is enabled for unprivileged users
JIT compiler kallsyms exports are enabled for root
Unable to retrieve global memory limit for JIT compiler for unprivileged users
CONFIG_BPF is set to y
CONFIG_BPF_SYSCALL is set to y
CONFIG_HAVE_EBPF_JIT is set to y
CONFIG_BPF_JIT is set to y
CONFIG_BPF_JIT_ALWAYS_ON is set to y
CONFIG_CGROUPS is set to y
CONFIG_CGROUP_BPF is set to y
CONFIG_CGROUP_NET_CLASSID is set to y
CONFIG_SOCK_CGROUP_DATA is set to y
CONFIG_BPF_EVENTS is set to y
CONFIG_KPROBE_EVENTS is set to y
CONFIG_UPROBE_EVENTS is set to y
CONFIG_TRACING is set to y
CONFIG_FTRACE_SYSCALLS is set to y
CONFIG_FUNCTION_ERROR_INJECTION is set to y
CONFIG_BPF_KPROBE_OVERRIDE is not set
CONFIG_NET is set to y
CONFIG_XDP_SOCKETS is set to y
CONFIG_LWTUNNEL_BPF is not set
CONFIG_NET_ACT_BPF is set to m
CONFIG_NET_CLS_BPF is set to m
CONFIG_NET_CLS_ACT is set to y
CONFIG_NET_SCH_INGRESS is set to m
CONFIG_XFRM is set to y
CONFIG_IP_ROUTE_CLASSID is set to y
CONFIG_IPV6_SEG6_BPF is not set
CONFIG_BPF_LIRC_MODE2 is not set
CONFIG_BPF_STREAM_PARSER is not set
CONFIG_NETFILTER_XT_MATCH_BPF is set to m
CONFIG_BPFILTER is not set
CONFIG_BPFILTER_UMH is not set
CONFIG_TEST_BPF is not set

Scanning system call availability...
bpf() syscall is available

Scanning eBPF program types...
eBPF program_type socket_filter is available
eBPF program_type kprobe is available
eBPF program_type sched_cls is available
eBPF program_type sched_act is available
eBPF program_type tracepoint is available
eBPF program_type xdp is available
eBPF program_type perf_event is available
eBPF program_type cgroup_skb is available
eBPF program_type cgroup_sock is available
eBPF program_type lwt_in is available
eBPF program_type lwt_out is available
eBPF program_type lwt_xmit is available
eBPF program_type sock_ops is available
eBPF program_type sk_skb is available
eBPF program_type cgroup_device is available
eBPF program_type sk_msg is available
eBPF program_type raw_tracepoint is available
eBPF program_type cgroup_sock_addr is available
eBPF program_type lwt_seg6local is available
eBPF program_type lirc_mode2 is NOT available
eBPF program_type sk_reuseport is NOT available
eBPF program_type flow_dissector is NOT available
eBPF program_type cgroup_sysctl is NOT available
eBPF program_type raw_tracepoint_writable is NOT available
eBPF program_type cgroup_sockopt is NOT available
eBPF program_type tracing is NOT available
eBPF program_type struct_ops is NOT available
eBPF program_type ext is NOT available

Scanning eBPF map types...
eBPF map_type hash is available
eBPF map_type array is available
eBPF map_type prog_array is available
eBPF map_type perf_event_array is available
eBPF map_type percpu_hash is available
eBPF map_type percpu_array is available
eBPF map_type stack_trace is available
eBPF map_type cgroup_array is available
eBPF map_type lru_hash is available
eBPF map_type lru_percpu_hash is available
eBPF map_type lpm_trie is available
eBPF map_type array_of_maps is available
eBPF map_type hash_of_maps is available
eBPF map_type devmap is available
eBPF map_type sockmap is NOT available
eBPF map_type cpumap is available
eBPF map_type xskmap is available
eBPF map_type sockhash is NOT available
eBPF map_type cgroup_storage is NOT available
eBPF map_type reuseport_sockarray is NOT available
eBPF map_type percpu_cgroup_storage is NOT available
eBPF map_type queue is NOT available
eBPF map_type stack is NOT available
eBPF map_type sk_storage is NOT available
eBPF map_type devmap_hash is NOT available
eBPF map_type struct_ops is NOT available

Scanning eBPF helper functions...
eBPF helpers supported for program type socket_filter:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_skb_load_bytes
	- bpf_get_numa_node_id
	- bpf_get_socket_cookie
	- bpf_get_socket_uid
	- bpf_skb_load_bytes_relative
eBPF helpers supported for program type kprobe:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_probe_read
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_current_pid_tgid
	- bpf_get_current_uid_gid
	- bpf_get_current_comm
	- bpf_perf_event_read
	- bpf_perf_event_output
	- bpf_get_stackid
	- bpf_get_current_task
	- bpf_current_task_under_cgroup
	- bpf_get_numa_node_id
	- bpf_probe_read_str
	- bpf_perf_event_read_value
	- bpf_get_stack
	- bpf_get_current_cgroup_id
eBPF helpers supported for program type sched_cls:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_skb_store_bytes
	- bpf_l3_csum_replace
	- bpf_l4_csum_replace
	- bpf_tail_call
	- bpf_clone_redirect
	- bpf_get_cgroup_classid
	- bpf_skb_vlan_push
	- bpf_skb_vlan_pop
	- bpf_skb_get_tunnel_key
	- bpf_skb_set_tunnel_key
	- bpf_redirect
	- bpf_get_route_realm
	- bpf_perf_event_output
	- bpf_skb_load_bytes
	- bpf_csum_diff
	- bpf_skb_get_tunnel_opt
	- bpf_skb_set_tunnel_opt
	- bpf_skb_change_proto
	- bpf_skb_change_type
	- bpf_skb_under_cgroup
	- bpf_get_hash_recalc
	- bpf_skb_change_tail
	- bpf_skb_pull_data
	- bpf_csum_update
	- bpf_set_hash_invalid
	- bpf_get_numa_node_id
	- bpf_get_socket_cookie
	- bpf_get_socket_uid
	- bpf_set_hash
	- bpf_skb_adjust_room
	- bpf_skb_get_xfrm_state
	- bpf_skb_load_bytes_relative
	- bpf_fib_lookup
	- bpf_skb_cgroup_id
eBPF helpers supported for program type sched_act:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_skb_store_bytes
	- bpf_l3_csum_replace
	- bpf_l4_csum_replace
	- bpf_tail_call
	- bpf_clone_redirect
	- bpf_get_cgroup_classid
	- bpf_skb_vlan_push
	- bpf_skb_vlan_pop
	- bpf_skb_get_tunnel_key
	- bpf_skb_set_tunnel_key
	- bpf_redirect
	- bpf_get_route_realm
	- bpf_perf_event_output
	- bpf_skb_load_bytes
	- bpf_csum_diff
	- bpf_skb_get_tunnel_opt
	- bpf_skb_set_tunnel_opt
	- bpf_skb_change_proto
	- bpf_skb_change_type
	- bpf_skb_under_cgroup
	- bpf_get_hash_recalc
	- bpf_skb_change_tail
	- bpf_skb_pull_data
	- bpf_csum_update
	- bpf_set_hash_invalid
	- bpf_get_numa_node_id
	- bpf_get_socket_cookie
	- bpf_get_socket_uid
	- bpf_set_hash
	- bpf_skb_adjust_room
	- bpf_skb_get_xfrm_state
	- bpf_skb_load_bytes_relative
	- bpf_fib_lookup
	- bpf_skb_cgroup_id
eBPF helpers supported for program type tracepoint:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_probe_read
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_current_pid_tgid
	- bpf_get_current_uid_gid
	- bpf_get_current_comm
	- bpf_perf_event_read
	- bpf_perf_event_output
	- bpf_get_stackid
	- bpf_get_current_task
	- bpf_current_task_under_cgroup
	- bpf_get_numa_node_id
	- bpf_probe_read_str
	- bpf_get_stack
	- bpf_get_current_cgroup_id
eBPF helpers supported for program type xdp:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_redirect
	- bpf_perf_event_output
	- bpf_csum_diff
	- bpf_get_numa_node_id
	- bpf_xdp_adjust_head
	- bpf_redirect_map
	- bpf_xdp_adjust_meta
	- bpf_xdp_adjust_tail
	- bpf_fib_lookup
eBPF helpers supported for program type perf_event:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_probe_read
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_current_pid_tgid
	- bpf_get_current_uid_gid
	- bpf_get_current_comm
	- bpf_perf_event_read
	- bpf_perf_event_output
	- bpf_get_stackid
	- bpf_get_current_task
	- bpf_current_task_under_cgroup
	- bpf_get_numa_node_id
	- bpf_probe_read_str
	- bpf_perf_prog_read_value
	- bpf_get_stack
	- bpf_get_current_cgroup_id
eBPF helpers supported for program type cgroup_skb:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_skb_load_bytes
	- bpf_get_numa_node_id
	- bpf_get_socket_cookie
	- bpf_get_socket_uid
	- bpf_skb_load_bytes_relative
eBPF helpers supported for program type cgroup_sock:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_current_uid_gid
	- bpf_get_numa_node_id
eBPF helpers supported for program type lwt_in:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_cgroup_classid
	- bpf_get_route_realm
	- bpf_perf_event_output
	- bpf_skb_load_bytes
	- bpf_csum_diff
	- bpf_skb_under_cgroup
	- bpf_get_hash_recalc
	- bpf_skb_pull_data
	- bpf_get_numa_node_id
	- bpf_lwt_push_encap
eBPF helpers supported for program type lwt_out:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_cgroup_classid
	- bpf_get_route_realm
	- bpf_perf_event_output
	- bpf_skb_load_bytes
	- bpf_csum_diff
	- bpf_skb_under_cgroup
	- bpf_get_hash_recalc
	- bpf_skb_pull_data
	- bpf_get_numa_node_id
eBPF helpers supported for program type lwt_xmit:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_skb_store_bytes
	- bpf_l3_csum_replace
	- bpf_l4_csum_replace
	- bpf_tail_call
	- bpf_clone_redirect
	- bpf_get_cgroup_classid
	- bpf_skb_get_tunnel_key
	- bpf_skb_set_tunnel_key
	- bpf_redirect
	- bpf_get_route_realm
	- bpf_perf_event_output
	- bpf_skb_load_bytes
	- bpf_csum_diff
	- bpf_skb_get_tunnel_opt
	- bpf_skb_set_tunnel_opt
	- bpf_skb_under_cgroup
	- bpf_get_hash_recalc
	- bpf_skb_change_tail
	- bpf_skb_pull_data
	- bpf_csum_update
	- bpf_set_hash_invalid
	- bpf_get_numa_node_id
	- bpf_skb_change_head
eBPF helpers supported for program type sock_ops:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_numa_node_id
	- bpf_setsockopt
	- bpf_sock_map_update
	- bpf_getsockopt
	- bpf_sock_ops_cb_flags_set
	- bpf_sock_hash_update
eBPF helpers supported for program type sk_skb:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_skb_store_bytes
	- bpf_tail_call
	- bpf_skb_load_bytes
	- bpf_skb_change_tail
	- bpf_skb_pull_data
	- bpf_get_numa_node_id
	- bpf_skb_change_head
	- bpf_get_socket_cookie
	- bpf_get_socket_uid
	- bpf_sk_redirect_map
	- bpf_sk_redirect_hash
eBPF helpers supported for program type cgroup_device:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_get_current_uid_gid
eBPF helpers supported for program type sk_msg:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_numa_node_id
	- bpf_msg_redirect_map
	- bpf_msg_apply_bytes
	- bpf_msg_cork_bytes
	- bpf_msg_pull_data
	- bpf_msg_redirect_hash
eBPF helpers supported for program type raw_tracepoint:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_probe_read
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_current_pid_tgid
	- bpf_get_current_uid_gid
	- bpf_get_current_comm
	- bpf_perf_event_read
	- bpf_perf_event_output
	- bpf_get_stackid
	- bpf_get_current_task
	- bpf_current_task_under_cgroup
	- bpf_get_numa_node_id
	- bpf_probe_read_str
	- bpf_get_stack
	- bpf_get_current_cgroup_id
eBPF helpers supported for program type cgroup_sock_addr:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_current_uid_gid
	- bpf_get_numa_node_id
	- bpf_bind
eBPF helpers supported for program type lwt_seg6local:
	- bpf_map_lookup_elem
	- bpf_map_update_elem
	- bpf_map_delete_elem
	- bpf_ktime_get_ns
	- bpf_get_prandom_u32
	- bpf_get_smp_processor_id
	- bpf_tail_call
	- bpf_get_cgroup_classid
	- bpf_get_route_realm
	- bpf_perf_event_output
	- bpf_skb_load_bytes
	- bpf_csum_diff
	- bpf_skb_under_cgroup
	- bpf_get_hash_recalc
	- bpf_skb_pull_data
	- bpf_get_numa_node_id
eBPF helpers supported for program type lirc_mode2:
eBPF helpers supported for program type sk_reuseport:
eBPF helpers supported for program type flow_dissector:
eBPF helpers supported for program type cgroup_sysctl:
eBPF helpers supported for program type raw_tracepoint_writable:
eBPF helpers supported for program type cgroup_sockopt:
eBPF helpers supported for program type tracing:
eBPF helpers supported for program type struct_ops:
eBPF helpers supported for program type ext:

Scanning miscellaneous eBPF features...
Large program size limit is NOT available


```