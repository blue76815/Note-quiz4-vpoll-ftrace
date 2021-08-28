# (總結)quiz3 vpoll  ftrace分析
###### tags: `linux-summer-2021`
<style>
.blue {
  color: blue; 
}
.red {
  color: red;
}
</style>
## 參考資料
[linux ftrace原理](https://www.geek-share.com/detail/2667839806.html)

[2.1 ftrace 的原理与使用](https://hotttao.github.io/2020/01/03/linux_perf/03_ftrace/)

## ftrace 分析流程

### 範例1.分析 `vpoll_poll()` 

先掛載模組
`sudo insmod vpoll.ko`

然後在執行 `./user` 檔時，用 ftrace 分析，紀錄 vpoll_poll() 在 kernel 內有執行那些 system call 函式

ftrace 分析步驟整理如下

剛開始使用 ftrace 時，先改成 root 權限，並把 ftrace 掛載到 debugfs 上，在終端機若想退出 root 權限，則輸入 exit 

```
sudo su
$ mount -t debugfs /sys/kernel/debug
```

```
step 0.切換到 ftrace 所在路徑 /sys/kernel/debug/tracing/
$ cd /sys/kernel/debug/tracing/

step 1. 清除 current_tracer
$ echo nop > current_tracer

step 2. 設置要顯示調用棧的函數
$ echo vpoll_poll > set_graph_function 

step 3. 配置跟蹤選項，開啟函式呼叫跟蹤，並跟蹤調用進程
$ echo function_graph > current_tracer
$ echo funcgraph-proc > trace_options

step 4. 開啟追蹤
$ echo 1 > tracing_on

step 5. 執行測試檔 ./user，再關閉追蹤：
$ /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
$ echo 0 > tracing_on

step 6.查看追蹤結果
$ cat trace
```
完整操作紀錄如下

```
blue76815@blue76815-virtual-machine:~$ sudo su
[sudo] password for blue76815: 
root@blue76815-virtual-machine:/home/blue76815# cd /sys/kernel/debug/tracing/
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo nop > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo vpoll_poll > set_graph_function 
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo function_graph > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo funcgraph-proc > trace_options
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 1 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
timeout...
GOT event 1
GOT event 1
timeout...
GOT event 3
GOT event 2
GOT event 4
timeout...
GOT event 10
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 0 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# cat trace
# tracer: function_graph
#
# CPU  TASK/PID         DURATION                  FUNCTION CALLS
# |     |    |           |   |                     |   |   |   |
   0)   user-29586   |               |  vpoll_poll [vpoll]() {
   0)   user-29586   |               |    ep_ptable_queue_proc() {
   0)   user-29586   |               |      kmem_cache_alloc() {
   0)   user-29586   |               |        _cond_resched() {
   0)   user-29586   |   0.227 us    |          rcu_all_qs();
   0)   user-29586   |   0.835 us    |        }
   0)   user-29586   |   0.192 us    |        should_failslab();
   0)   user-29586   |   1.970 us    |      }
   0)   user-29586   |               |      add_wait_queue() {
   0)   user-29586   |   0.192 us    |        _raw_spin_lock_irqsave();
   0)   user-29586   |   0.202 us    |        _raw_spin_unlock_irqrestore();
   0)   user-29586   |   0.975 us    |      }
   0)   user-29586   |   3.809 us    |    }
   0)   user-29586   |   6.365 us    |  }
   0)   user-29586   |   3.723 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   0.923 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   2.708 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   0.865 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   3.811 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   0.754 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   2.581 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   0.873 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   1.501 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   0.484 us    |  vpoll_poll [vpoll]();
   0)   user-29586   |   2.245 us    |  vpoll_poll [vpoll]();
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# 
```
![](https://i.imgur.com/vzZfGLn.png)

### 範例2.分析 `vpoll_ioctl() `
同樣維持在 root 權限
ftrace 所在路徑 /sys/kernel/debug/tracing/ 下操作

```
step 1. 清除 current_tracer
$ echo nop > current_tracer


step 2. 設置要顯示調用棧的函數
$ echo vpoll_ioctl > set_graph_function

step 3. 配置跟蹤選項，開啟函式呼叫跟蹤，並跟蹤調用進程
$ echo function_graph > current_tracer
$ echo funcgraph-proc > trace_options

step 4. 開啟追蹤
$ echo 1 > tracing_on

step 5. 執行測試檔 ./user，再關閉追蹤：
$ /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
$ echo 0 > tracing_on

step 6.查看追蹤結果
$ cat trace
```
完整記錄

```
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo nop > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo vpoll_ioctl > set_graph_function
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo function_graph > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo funcgraph-proc > trace_options
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 1 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
timeout...
GOT event 1
GOT event 1
timeout...
GOT event 3
GOT event 2
timeout...
GOT event 4
timeout...
GOT event 10
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 0 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# cat trace
# tracer: function_graph
#
# CPU  TASK/PID         DURATION                  FUNCTION CALLS
# |     |    |           |   |                     |   |   |   |
   0)   user-29695   |               |  vpoll_ioctl [vpoll]() {
   0)   user-29695   |   1.561 us    |    _raw_spin_lock_irq();
   0)   user-29695   |               |    __wake_up_locked_key() {
   0)   user-29695   |               |      __wake_up_common() {
   0)   user-29695   |               |        ep_poll_callback() {
   0)   user-29695   |   1.419 us    |          _raw_read_lock_irqsave();
   0)   user-29695   |   0.447 us    |          _raw_read_unlock_irqrestore();
   0)   user-29695   |   3.451 us    |        }
   0)   user-29695   |   5.333 us    |      }
   0)   user-29695   |   6.180 us    |    }
   0)   user-29695   | + 12.428 us   |  }
   1)   user-29694   |               |  vpoll_ioctl [vpoll]() {
   1)   user-29694   |   0.804 us    |    _raw_spin_lock_irq();
   1)   user-29694   |               |    __wake_up_locked_key() {
   1)   user-29694   |               |      __wake_up_common() {
   1)   user-29694   |               |        ep_poll_callback() {
   1)   user-29694   |   0.424 us    |          _raw_read_lock_irqsave();
   1)   user-29694   |   0.437 us    |          _raw_read_unlock_irqrestore();
   1)   user-29694   |   2.282 us    |        }
   1)   user-29694   |   3.728 us    |      }
   1)   user-29694   |   4.535 us    |    }
   1)   user-29694   |   8.944 us    |  }
   0)   user-29695   |               |  vpoll_ioctl [vpoll]() {
   0)   user-29695   |   1.819 us    |    _raw_spin_lock_irq();
   0)   user-29695   |               |    __wake_up_locked_key() {
   0)   user-29695   |               |      __wake_up_common() {
   0)   user-29695   |               |        ep_poll_callback() {
   0)   user-29695   |   1.537 us    |          _raw_read_lock_irqsave();
   0)   user-29695   |               |          __wake_up() {
   0)   user-29695   |               |            __wake_up_common_lock() {
   0)   user-29695   |   0.490 us    |              _raw_spin_lock_irqsave();
   0)   user-29695   |               |              __wake_up_common() {
   0)   user-29695   |               |                autoremove_wake_function() {
   0)   user-29695   |               |                  default_wake_function() {
   0)   user-29695   |               |                    try_to_wake_up() {
   0)   user-29695   |   1.067 us    |                      _raw_spin_lock_irqsave();
   0)   user-29695   |               |                      select_task_rq_fair() {
   0)   user-29695   |   0.528 us    |                        available_idle_cpu();
   0)   user-29695   |   2.629 us    |                        update_cfs_rq_h_load();
   0)   user-29695   |               |                        select_idle_sibling() {
   0)   user-29695   |   0.568 us    |                          available_idle_cpu();
   0)   user-29695   |   1.545 us    |                        }
   0)   user-29695   |   7.275 us    |                      }
   0)   user-29695   |               |                      native_smp_send_reschedule() {
   0)   user-29695   | + 41.452 us   |                        x2apic_send_IPI();
   0)   user-29695   | + 43.869 us   |                      }
   0)   user-29695   |   0.876 us    |                      _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 57.322 us   |                    }
   0)   user-29695   | + 58.338 us   |                  }
   0)   user-29695   | + 59.707 us   |                }
   0)   user-29695   | + 61.234 us   |              }
   0)   user-29695   |   0.489 us    |              _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 64.224 us   |            }
   0)   user-29695   | + 65.168 us   |          }
   0)   user-29695   |   0.533 us    |          _raw_read_unlock_irqrestore();
   0)   user-29695   | + 69.611 us   |        }
   0)   user-29695   | + 71.605 us   |      }
   0)   user-29695   | + 72.712 us   |    }
   0)   user-29695   | + 79.850 us   |  }
   1)   user-29694   |               |  vpoll_ioctl [vpoll]() {
   1)   user-29694   |   1.061 us    |    _raw_spin_lock_irq();
   1)   user-29694   |               |    __wake_up_locked_key() {
   1)   user-29694   |               |      __wake_up_common() {
   1)   user-29694   |               |        ep_poll_callback() {
   1)   user-29694   |   0.864 us    |          _raw_read_lock_irqsave();
   1)   user-29694   |   0.550 us    |          _raw_read_unlock_irqrestore();
   1)   user-29694   |   3.154 us    |        }
   1)   user-29694   |   5.173 us    |      }
   1)   user-29694   |   6.182 us    |    }
   1)   user-29694   | + 11.879 us   |  }
   0)   user-29695   |               |  vpoll_ioctl [vpoll]() {
   0)   user-29695   |   1.493 us    |    _raw_spin_lock_irq();
   0)   user-29695   |               |    __wake_up_locked_key() {
   0)   user-29695   |               |      __wake_up_common() {
   0)   user-29695   |               |        ep_poll_callback() {
   0)   user-29695   |   1.847 us    |          _raw_read_lock_irqsave();
   0)   user-29695   |               |          __wake_up() {
   0)   user-29695   |               |            __wake_up_common_lock() {
   0)   user-29695   |   0.444 us    |              _raw_spin_lock_irqsave();
   0)   user-29695   |               |              __wake_up_common() {
   0)   user-29695   |               |                autoremove_wake_function() {
   0)   user-29695   |               |                  default_wake_function() {
   0)   user-29695   |               |                    try_to_wake_up() {
   0)   user-29695   |   0.956 us    |                      _raw_spin_lock_irqsave();
   0)   user-29695   |   0.472 us    |                      _raw_spin_unlock_irqrestore();
   0)   user-29695   |   2.987 us    |                    }
   0)   user-29695   |   3.861 us    |                  }
   0)   user-29695   |   4.752 us    |                }
   0)   user-29695   |   6.172 us    |              }
   0)   user-29695   |   0.446 us    |              _raw_spin_unlock_irqrestore();
   0)   user-29695   |   8.807 us    |            }
   0)   user-29695   |   9.651 us    |          }
   0)   user-29695   |   0.462 us    |          _raw_read_unlock_irqrestore();
   0)   user-29695   | + 14.083 us   |        }
   0)   user-29695   | + 15.994 us   |      }
   0)   user-29695   | + 16.917 us   |    }
   0)   user-29695   | + 23.587 us   |  }
   1)   user-29694   |               |  vpoll_ioctl [vpoll]() {
   1)   user-29694   |   0.852 us    |    _raw_spin_lock_irq();
   1)   user-29694   |               |    __wake_up_locked_key() {
   1)   user-29694   |               |      __wake_up_common() {
   1)   user-29694   |               |        ep_poll_callback() {
   1)   user-29694   |   0.446 us    |          _raw_read_lock_irqsave();
   1)   user-29694   |   0.478 us    |          _raw_read_unlock_irqrestore();
   1)   user-29694   |   2.459 us    |        }
   1)   user-29694   |   3.827 us    |      }
   1)   user-29694   |   4.730 us    |    }
   1)   user-29694   |   9.523 us    |  }
   0)   user-29695   |               |  vpoll_ioctl [vpoll]() {
   0)   user-29695   |   1.079 us    |    _raw_spin_lock_irq();
   0)   user-29695   |               |    __wake_up_locked_key() {
   0)   user-29695   |               |      __wake_up_common() {
   0)   user-29695   |               |        ep_poll_callback() {
   0)   user-29695   |   0.729 us    |          _raw_read_lock_irqsave();
   0)   user-29695   |               |          __wake_up() {
   0)   user-29695   |               |            __wake_up_common_lock() {
   0)   user-29695   |   0.132 us    |              _raw_spin_lock_irqsave();
   0)   user-29695   |               |              __wake_up_common() {
   0)   user-29695   |               |                autoremove_wake_function() {
   0)   user-29695   |               |                  default_wake_function() {
   0)   user-29695   |               |                    try_to_wake_up() {
   0)   user-29695   |   0.688 us    |                      _raw_spin_lock_irqsave();
   0)   user-29695   |               |                      select_task_rq_fair() {
   0)   user-29695   |   0.145 us    |                        available_idle_cpu();
   0)   user-29695   |   1.018 us    |                        update_cfs_rq_h_load();
   0)   user-29695   |               |                        select_idle_sibling() {
   0)   user-29695   |   0.554 us    |                          available_idle_cpu();
   0)   user-29695   |   0.828 us    |                        }
   0)   user-29695   |   3.250 us    |                      }
   0)   user-29695   |               |                      native_smp_send_reschedule() {
   0)   user-29695   | + 14.335 us   |                        x2apic_send_IPI();
   0)   user-29695   | + 15.067 us   |                      }
   0)   user-29695   |   0.231 us    |                      _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 20.626 us   |                    }
   0)   user-29695   | + 20.917 us   |                  }
   0)   user-29695   | + 21.263 us   |                }
   0)   user-29695   | + 21.957 us   |              }
   0)   user-29695   |   0.133 us    |              _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 22.750 us   |            }
   0)   user-29695   | + 23.001 us   |          }
   0)   user-29695   |   0.140 us    |          _raw_read_unlock_irqrestore();
   0)   user-29695   | + 24.676 us   |        }
   0)   user-29695   | + 25.593 us   |      }
   0)   user-29695   | + 25.933 us   |    }
   0)   user-29695   | + 29.198 us   |  }
   1)   user-29694   |               |  vpoll_ioctl [vpoll]() {
   1)   user-29694   |   0.261 us    |    _raw_spin_lock_irq();
   1)   user-29694   |               |    __wake_up_locked_key() {
   1)   user-29694   |               |      __wake_up_common() {
   1)   user-29694   |               |        ep_poll_callback() {
   1)   user-29694   |   0.142 us    |          _raw_read_lock_irqsave();
   1)   user-29694   |   0.144 us    |          _raw_read_unlock_irqrestore();
   1)   user-29694   |   0.727 us    |        }
   1)   user-29694   |   1.139 us    |      }
   1)   user-29694   |   1.408 us    |    }
   1)   user-29694   |   2.933 us    |  }
   0)   user-29695   |               |  vpoll_ioctl [vpoll]() {
   0)   user-29695   |   1.887 us    |    _raw_spin_lock_irq();
   0)   user-29695   |               |    __wake_up_locked_key() {
   0)   user-29695   |               |      __wake_up_common() {
   0)   user-29695   |               |        ep_poll_callback() {
   0)   user-29695   |   1.580 us    |          _raw_read_lock_irqsave();
   0)   user-29695   |               |          __wake_up() {
   0)   user-29695   |               |            __wake_up_common_lock() {
   0)   user-29695   |   0.479 us    |              _raw_spin_lock_irqsave();
   0)   user-29695   |               |              __wake_up_common() {
   0)   user-29695   |               |                autoremove_wake_function() {
   0)   user-29695   |               |                  default_wake_function() {
   0)   user-29695   |               |                    try_to_wake_up() {
   0)   user-29695   |   1.167 us    |                      _raw_spin_lock_irqsave();
   0)   user-29695   |               |                      select_task_rq_fair() {
   0)   user-29695   |   0.532 us    |                        available_idle_cpu();
   0)   user-29695   |   2.307 us    |                        update_cfs_rq_h_load();
   0)   user-29695   |               |                        select_idle_sibling() {
   0)   user-29695   |   0.571 us    |                          available_idle_cpu();
   0)   user-29695   |   1.562 us    |                        }
   0)   user-29695   |   6.972 us    |                      }
   0)   user-29695   |               |                      native_smp_send_reschedule() {
   0)   user-29695   | + 45.902 us   |                        x2apic_send_IPI();
   0)   user-29695   | + 48.364 us   |                      }
   0)   user-29695   |   0.897 us    |                      _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 61.515 us   |                    }
   0)   user-29695   | + 62.551 us   |                  }
   0)   user-29695   | + 63.880 us   |                }
   0)   user-29695   | + 65.281 us   |              }
   0)   user-29695   |   0.500 us    |              _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 68.224 us   |            }
   0)   user-29695   | + 69.147 us   |          }
   0)   user-29695   |   0.542 us    |          _raw_read_unlock_irqrestore();
   0)   user-29695   | + 73.727 us   |        }
   0)   user-29695   | + 75.739 us   |      }
   0)   user-29695   | + 76.885 us   |    }
   0)   user-29695   |   ==========> |
   0)   user-29695   |               |    smp_apic_timer_interrupt() {
   0)   user-29695   |               |      irq_enter() {
   0)   user-29695   |   0.582 us    |        rcu_irq_enter();
   0)   user-29695   |   1.577 us    |      }
   0)   user-29695   |               |      hrtimer_interrupt() {
   0)   user-29695   |   0.609 us    |        _raw_spin_lock_irqsave();
   0)   user-29695   |   1.034 us    |        ktime_get_update_offsets_now();
   0)   user-29695   |               |        __hrtimer_run_queues() {
   0)   user-29695   |   0.858 us    |          __remove_hrtimer();
   0)   user-29695   |   0.550 us    |          _raw_spin_unlock_irqrestore();
   0)   user-29695   |               |          tick_sched_timer() {
   0)   user-29695   |   0.617 us    |            ktime_get();
   0)   user-29695   |               |            tick_sched_do_timer() {
   0)   user-29695   |               |              tick_do_update_jiffies64.part.14() {
   0)   user-29695   |   0.546 us    |                _raw_spin_lock();
   0)   user-29695   |               |                do_timer() {
   0)   user-29695   |   0.617 us    |                  calc_global_load();
   0)   user-29695   |   1.541 us    |                }
   0)   user-29695   |               |                update_wall_time() {
   0)   user-29695   |               |                  timekeeping_advance() {
   0)   user-29695   |   0.497 us    |                    _raw_spin_lock_irqsave();
   0)   user-29695   |   0.500 us    |                    _raw_spin_unlock_irqrestore();
   0)   user-29695   |   2.595 us    |                  }
   0)   user-29695   |   3.538 us    |                }
   0)   user-29695   |   7.548 us    |              }
   1)   user-29694   |               |  vpoll_ioctl [vpoll]() {
   1)   user-29694   |   1.186 us    |    _raw_spin_lock_irq();
   1)   user-29694   |               |    __wake_up_locked_key() {
   1)   user-29694   |               |      __wake_up_common() {
   1)   user-29694   |               |        ep_poll_callback() {
   1)   user-29694   |   0.967 us    |          _raw_read_lock_irqsave();
   1)   user-29694   |   0.539 us    |          _raw_read_unlock_irqrestore();
   1)   user-29694   |   3.229 us    |        }
   1)   user-29694   |   4.958 us    |      }
   1)   user-29694   |   5.960 us    |    }
   1)   user-29694   | + 11.834 us   |  }
   0)   user-29695   |   8.593 us    |            }
   0)   user-29695   |               |            tick_sched_handle() {
   0)   user-29695   |               |              update_process_times() {
   0)   user-29695   |               |                account_process_tick() {
   0)   user-29695   |               |                  account_system_time() {
   0)   user-29695   |               |                    account_system_index_time() {
   0)   user-29695   |   3.966 us    |                      cpuacct_account_field();
   0)   user-29695   |               |                      __cgroup_account_cputime_field() {
   0)   user-29695   |   0.578 us    |                        cgroup_rstat_updated();
   0)   user-29695   |   2.156 us    |                      }
   0)   user-29695   |               |                      acct_account_cputime() {
   0)   user-29695   |   1.195 us    |                        __acct_update_integrals();
   0)   user-29695   |   2.162 us    |                      }
   0)   user-29695   | + 11.136 us   |                    }
   0)   user-29695   | + 12.171 us   |                  }
   0)   user-29695   | + 13.391 us   |                }
   0)   user-29695   |               |                run_local_timers() {
   0)   user-29695   |   0.651 us    |                  hrtimer_run_queues();
   0)   user-29695   |   0.651 us    |                  raise_softirq();
   0)   user-29695   |   3.011 us    |                }
   0)   user-29695   |               |                rcu_sched_clock_irq() {
   0)   user-29695   |   0.554 us    |                  rcu_segcblist_ready_cbs();
   0)   user-29695   |   2.255 us    |                }
   0)   user-29695   |               |                scheduler_tick() {
   0)   user-29695   |   0.524 us    |                  _raw_spin_lock();
   0)   user-29695   |               |                  update_rq_clock() {
   0)   user-29695   |   0.548 us    |                    update_rq_clock.part.82();
   0)   user-29695   |   1.534 us    |                  }
   0)   user-29695   |               |                  task_tick_fair() {
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   1.080 us    |                      update_min_vruntime();
   0)   user-29695   |   0.989 us    |                      cpuacct_charge();
   0)   user-29695   |               |                      __cgroup_account_cputime() {
   0)   user-29695   |   0.473 us    |                        cgroup_rstat_updated();
   0)   user-29695   |   1.421 us    |                      }
   0)   user-29695   |   5.673 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_se() {
   0)   user-29695   |   0.536 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.682 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.489 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.573 us    |                    }
   0)   user-29695   |   0.451 us    |                    update_cfs_group();
   0)   user-29695   |   0.516 us    |                    hrtimer_active();
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   1.151 us    |                      update_min_vruntime();
   0)   user-29695   |   2.221 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_se() {
   0)   user-29695   |   0.493 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.576 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.485 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.516 us    |                    }
   0)   user-29695   |               |                    update_cfs_group() {
   0)   user-29695   |               |                      reweight_entity() {
   0)   user-29695   |   0.516 us    |                        update_curr();
   0)   user-29695   |   0.475 us    |                        account_entity_dequeue();
   0)   user-29695   |   0.505 us    |                        account_entity_enqueue();
   0)   user-29695   |   3.898 us    |                      }
   0)   user-29695   |   4.935 us    |                    }
   0)   user-29695   |   0.480 us    |                    hrtimer_active();
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   0.481 us    |                      update_min_vruntime();
   0)   user-29695   |   2.384 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_se() {
   0)   user-29695   |   0.507 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.556 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.480 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.515 us    |                    }
   0)   user-29695   |               |                    update_cfs_group() {
   0)   user-29695   |               |                      reweight_entity() {
   0)   user-29695   |   0.514 us    |                        update_curr();
   0)   user-29695   |   0.473 us    |                        account_entity_dequeue();
   0)   user-29695   |   0.484 us    |                        account_entity_enqueue();
   0)   user-29695   |   3.446 us    |                      }
   0)   user-29695   |   4.449 us    |                    }
   0)   user-29695   |   0.469 us    |                    hrtimer_active();
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   0.729 us    |                      update_min_vruntime();
   0)   user-29695   |   1.761 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_se() {
   0)   user-29695   |   0.488 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.572 us    |                    }
   0)   user-29695   |               |                    __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.542 us    |                      __accumulate_pelt_segments();
   0)   user-29695   |   1.663 us    |                    }
   0)   user-29695   |               |                    update_cfs_group() {
   0)   user-29695   |               |                      reweight_entity() {
   0)   user-29695   |   0.501 us    |                        update_curr();
   0)   user-29695   |   0.505 us    |                        account_entity_dequeue();
   0)   user-29695   |   0.506 us    |                        account_entity_enqueue();
   0)   user-29695   |   3.646 us    |                      }
   0)   user-29695   |   4.719 us    |                    }
   0)   user-29695   |   0.506 us    |                    hrtimer_active();
   0)   user-29695   | + 54.143 us   |                  }
   0)   user-29695   |   0.644 us    |                  calc_global_load_tick();
   0)   user-29695   |               |                  trigger_load_balance() {
   0)   user-29695   |   0.561 us    |                    raise_softirq();
   0)   user-29695   |   0.887 us    |                    nohz_balance_exit_idle();
   0)   user-29695   |   3.064 us    |                  }
   0)   user-29695   | + 64.009 us   |                }
   0)   user-29695   |   0.652 us    |                run_posix_cpu_timers();
   0)   user-29695   | + 87.541 us   |              }
   0)   user-29695   |   0.760 us    |              profile_tick();
   0)   user-29695   | + 91.189 us   |            }
   0)   user-29695   |   0.638 us    |            hrtimer_forward();
   0)   user-29695   | * 15028.28 us |          }
   0)   user-29695   |   0.598 us    |          _raw_spin_lock_irq();
   0)   user-29695   |   3.398 us    |          enqueue_hrtimer();
   0)   user-29695   | * 15037.74 us |        }
   0)   user-29695   |               |        hrtimer_update_next_event() {
   0)   user-29695   |               |          __hrtimer_get_next_event() {
   0)   user-29695   |   0.544 us    |            __hrtimer_next_event_base();
   0)   user-29695   |   1.639 us    |          }
   0)   user-29695   |               |          __hrtimer_get_next_event() {
   0)   user-29695   |   0.652 us    |            __hrtimer_next_event_base();
   0)   user-29695   |   1.696 us    |          }
   0)   user-29695   |   4.908 us    |        }
   0)   user-29695   |   0.528 us    |        _raw_spin_unlock_irqrestore();
   0)   user-29695   |               |        tick_program_event() {
   0)   user-29695   |               |          clockevents_program_event() {
   0)   user-29695   |   0.933 us    |            ktime_get();
   0)   user-29695   |   2.146 us    |          }
   0)   user-29695   |   3.220 us    |        }
   0)   user-29695   |   0.500 us    |        _raw_spin_lock_irqsave();
   0)   user-29695   |   0.713 us    |        ktime_get_update_offsets_now();
   0)   user-29695   |               |        __hrtimer_run_queues() {
   0)   user-29695   |   0.934 us    |          __remove_hrtimer();
   0)   user-29695   |   0.499 us    |          _raw_spin_unlock_irqrestore();
   0)   user-29695   |               |          tick_sched_timer() {
   0)   user-29695   |   0.569 us    |            ktime_get();
   0)   user-29695   |               |            tick_sched_do_timer() {
   0)   user-29695   |               |              tick_do_update_jiffies64.part.14() {
   0)   user-29695   |   0.715 us    |                _raw_spin_lock();
   0)   user-29695   |               |                do_timer() {
   0)   user-29695   |   0.656 us    |                  calc_global_load();
   0)   user-29695   |   1.647 us    |                }
   0)   user-29695   |               |                update_wall_time() {
   0)   user-29695   |               |                  timekeeping_advance() {
   0)   user-29695   |   0.614 us    |                    _raw_spin_lock_irqsave();
   0)   user-29695   |   0.535 us    |                    ntp_tick_length();
   0)   user-29695   |   0.500 us    |                    ntp_tick_length();
   0)   user-29695   |               |                    timekeeping_update() {
   0)   user-29695   |   0.536 us    |                      ntp_get_next_leap();
   0)   user-29695   |   0.818 us    |                      update_vsyscall();
   0)   user-29695   |               |                      raw_notifier_call_chain() {
   0)   user-29695   |   0.509 us    |                        notifier_call_chain();
   0)   user-29695   |   1.500 us    |                      }
   0)   user-29695   |   0.566 us    |                      update_fast_timekeeper();
   0)   user-29695   |   0.535 us    |                      update_fast_timekeeper();
   0)   user-29695   |   7.203 us    |                    }
   0)   user-29695   |   0.583 us    |                    _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 13.042 us   |                  }
   0)   user-29695   | + 14.134 us   |                }
   0)   user-29695   | + 18.525 us   |              }
   0)   user-29695   | + 19.634 us   |            }
   0)   user-29695   |               |            tick_sched_handle() {
   0)   user-29695   |               |              update_process_times() {
   0)   user-29695   |               |                account_process_tick() {
   0)   user-29695   |               |                  account_system_time() {
   0)   user-29695   |               |                    account_system_index_time() {
   0)   user-29695   |   0.591 us    |                      cpuacct_account_field();
   0)   user-29695   |               |                      __cgroup_account_cputime_field() {
   0)   user-29695   |   0.469 us    |                        cgroup_rstat_updated();
   0)   user-29695   |   1.390 us    |                      }
   0)   user-29695   |               |                      acct_account_cputime() {
   0)   user-29695   |   0.463 us    |                        __acct_update_integrals();
   0)   user-29695   |   1.365 us    |                      }
   0)   user-29695   |   6.275 us    |                    }
   0)   user-29695   |   7.270 us    |                  }
   0)   user-29695   |   8.223 us    |                }
   0)   user-29695   |               |                run_local_timers() {
   0)   user-29695   |   0.458 us    |                  hrtimer_run_queues();
   0)   user-29695   |   0.504 us    |                  raise_softirq();
   0)   user-29695   |   2.400 us    |                }
   0)   user-29695   |               |                rcu_sched_clock_irq() {
   0)   user-29695   |   0.462 us    |                  rcu_segcblist_ready_cbs();
   0)   user-29695   |   1.462 us    |                }
   0)   user-29695   |               |                scheduler_tick() {
   0)   user-29695   |   0.473 us    |                  _raw_spin_lock();
   0)   user-29695   |               |                  update_rq_clock() {
   0)   user-29695   |   0.489 us    |                    update_rq_clock.part.82();
   0)   user-29695   |   1.413 us    |                  }
   0)   user-29695   |               |                  task_tick_fair() {
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   0.483 us    |                      update_min_vruntime();
   0)   user-29695   |   0.554 us    |                      cpuacct_charge();
   0)   user-29695   |               |                      __cgroup_account_cputime() {
   0)   user-29695   |   0.541 us    |                        cgroup_rstat_updated();
   0)   user-29695   |   1.483 us    |                      }
   0)   user-29695   |   4.509 us    |                    }
   0)   user-29695   |   0.582 us    |                    __update_load_avg_se();
   0)   user-29695   |   0.654 us    |                    __update_load_avg_cfs_rq();
   0)   user-29695   |   0.469 us    |                    update_cfs_group();
   0)   user-29695   |   0.478 us    |                    hrtimer_active();
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   0.590 us    |                      __calc_delta();
   0)   user-29695   |   0.489 us    |                      update_min_vruntime();
   0)   user-29695   |   2.713 us    |                    }
   0)   user-29695   |   0.510 us    |                    __update_load_avg_se();
   0)   user-29695   |   0.492 us    |                    __update_load_avg_cfs_rq();
   0)   user-29695   |               |                    update_cfs_group() {
   0)   user-29695   |               |                      reweight_entity() {
   0)   user-29695   |   0.501 us    |                        update_curr();
   0)   user-29695   |   0.464 us    |                        account_entity_dequeue();
   0)   user-29695   |   0.463 us    |                        account_entity_enqueue();
   0)   user-29695   |   3.397 us    |                      }
   0)   user-29695   |   4.433 us    |                    }
   0)   user-29695   |   0.459 us    |                    hrtimer_active();
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   0.486 us    |                      update_min_vruntime();
   0)   user-29695   |   1.512 us    |                    }
   0)   user-29695   |   0.491 us    |                    __update_load_avg_se();
   0)   user-29695   |   0.541 us    |                    __update_load_avg_cfs_rq();
   0)   user-29695   |               |                    update_cfs_group() {
   0)   user-29695   |               |                      reweight_entity() {
   0)   user-29695   |   0.474 us    |                        update_curr();
   0)   user-29695   |   0.485 us    |                        account_entity_dequeue();
   0)   user-29695   |   0.478 us    |                        account_entity_enqueue();
   0)   user-29695   |   3.416 us    |                      }
   0)   user-29695   |   4.423 us    |                    }
   0)   user-29695   |   0.562 us    |                    hrtimer_active();
   0)   user-29695   |               |                    update_curr() {
   0)   user-29695   |   0.484 us    |                      update_min_vruntime();
   0)   user-29695   |   1.454 us    |                    }
   0)   user-29695   |   0.489 us    |                    __update_load_avg_se();
   0)   user-29695   |   0.500 us    |                    __update_load_avg_cfs_rq();
   0)   user-29695   |               |                    update_cfs_group() {
   0)   user-29695   |               |                      reweight_entity() {
   0)   user-29695   |   0.499 us    |                        update_curr();
   0)   user-29695   |   0.484 us    |                        account_entity_dequeue();
   0)   user-29695   |   0.482 us    |                        account_entity_enqueue();
   0)   user-29695   |   3.473 us    |                      }
   0)   user-29695   |   4.504 us    |                    }
   0)   user-29695   |   0.473 us    |                    hrtimer_active();
   0)   user-29695   | + 40.616 us   |                  }
   0)   user-29695   |   0.471 us    |                  calc_global_load_tick();
   0)   user-29695   |               |                  trigger_load_balance() {
   0)   user-29695   |   0.516 us    |                    raise_softirq();
   0)   user-29695   |   0.505 us    |                    nohz_balance_exit_idle();
   0)   user-29695   |   2.476 us    |                  }
   0)   user-29695   | + 48.682 us   |                }
   0)   user-29695   |   0.474 us    |                run_posix_cpu_timers();
   0)   user-29695   | + 64.146 us   |              }
   0)   user-29695   |   0.506 us    |              profile_tick();
   0)   user-29695   | + 66.124 us   |            }
   0)   user-29695   |   0.604 us    |            hrtimer_forward();
   0)   user-29695   | + 89.446 us   |          }
   0)   user-29695   |   0.478 us    |          _raw_spin_lock_irq();
   0)   user-29695   |   0.546 us    |          enqueue_hrtimer();
   0)   user-29695   | + 94.967 us   |        }
   0)   user-29695   |               |        hrtimer_update_next_event() {
   0)   user-29695   |               |          __hrtimer_get_next_event() {
   0)   user-29695   |   0.468 us    |            __hrtimer_next_event_base();
   0)   user-29695   |   1.494 us    |          }
   0)   user-29695   |               |          __hrtimer_get_next_event() {
   0)   user-29695   |   0.531 us    |            __hrtimer_next_event_base();
   0)   user-29695   |   1.483 us    |          }
   0)   user-29695   |   4.385 us    |        }
   0)   user-29695   |   0.480 us    |        _raw_spin_unlock_irqrestore();
   0)   user-29695   |               |        tick_program_event() {
   0)   user-29695   |               |          clockevents_program_event() {
   0)   user-29695   |   0.562 us    |            ktime_get();
   0)   user-29695   |   6.954 us    |            lapic_next_deadline();
   0)   user-29695   |   9.366 us    |          }
   0)   user-29695   | + 10.307 us   |        }
   0)   user-29695   | * 15166.26 us |      }
   0)   user-29695   |               |      irq_exit() {
   0)   user-29695   |   0.840 us    |        ksoftirqd_running();
   0)   user-29695   |               |        __do_softirq() {
   0)   user-29695   |               |          run_timer_softirq() {
   0)   user-29695   |   0.497 us    |            _raw_spin_lock_irq();
   0)   user-29695   |   1.202 us    |            __next_timer_interrupt();
   0)   user-29695   |   0.479 us    |            _raw_spin_lock_irq();
   0)   user-29695   |   0.901 us    |            __next_timer_interrupt();
   0)   user-29695   |   5.803 us    |          }
   0)   user-29695   |               |          run_rebalance_domains() {
   0)   user-29695   |               |            update_blocked_averages() {
   0)   user-29695   |   0.469 us    |              _raw_spin_lock_irqsave();
   0)   user-29695   |               |              update_rq_clock() {
   0)   user-29695   |   0.548 us    |                update_rq_clock.part.82();
   0)   user-29695   |   1.485 us    |              }
   0)   user-29695   |               |              update_rt_rq_load_avg() {
   0)   user-29695   |   0.478 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.649 us    |              }
   0)   user-29695   |               |              update_dl_rq_load_avg() {
   0)   user-29695   |   0.521 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.754 us    |              }
   0)   user-29695   |               |              __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.485 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.527 us    |              }
   0)   user-29695   |               |              __update_load_avg_se() {
   0)   user-29695   |   0.477 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.563 us    |              }
   0)   user-29695   |               |              __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.479 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.507 us    |              }
   0)   user-29695   |   0.513 us    |              __update_load_avg_cfs_rq();
   0)   user-29695   |               |              __update_load_avg_se() {
   0)   user-29695   |   0.484 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.523 us    |              }
   0)   user-29695   |               |              __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.474 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.572 us    |              }
   0)   user-29695   |               |              __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.473 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.791 us    |              }
   0)   user-29695   |               |              __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.474 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   2.039 us    |              }
   0)   user-29695   |               |              __update_load_avg_se() {
   0)   user-29695   |   0.480 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.753 us    |              }
   0)   user-29695   |               |              __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.478 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   1.616 us    |              }
   0)   user-29695   |               |              __update_load_avg_cfs_rq() {
   0)   user-29695   |   0.483 us    |                __accumulate_pelt_segments();
   0)   user-29695   | + 78.141 us   |              }
   0)   user-29695   |   1.065 us    |              __update_load_avg_cfs_rq();
   0)   user-29695   |               |              __update_load_avg_se() {
   0)   user-29695   |   0.491 us    |                __accumulate_pelt_segments();
   0)   user-29695   |   2.827 us    |              }
   0)   user-29695   |   0.537 us    |              __update_load_avg_cfs_rq();
   0)   user-29695   |   0.515 us    |              __update_load_avg_cfs_rq();
   0)   user-29695   |   0.500 us    |              _raw_spin_unlock_irqrestore();
   0)   user-29695   | ! 120.510 us  |            }
   0)   user-29695   |               |            rebalance_domains() {
   0)   user-29695   |   0.485 us    |              __msecs_to_jiffies();
   0)   user-29695   |               |              load_balance() {
   0)   user-29695   |   0.476 us    |                idle_cpu();
   0)   user-29695   |   0.475 us    |                group_balance_cpu();
   0)   user-29695   |               |                find_busiest_group() {
   0)   user-29695   |               |                  update_group_capacity() {
   0)   user-29695   |   0.477 us    |                    __msecs_to_jiffies();
   0)   user-29695   |   1.588 us    |                  }
   0)   user-29695   |   3.814 us    |                }
   0)   user-29695   |   7.968 us    |              }
   0)   user-29695   |   0.478 us    |              __msecs_to_jiffies();
   0)   user-29695   | + 11.474 us   |            }
   0)   user-29695   | ! 133.700 us  |          }
   0)   user-29695   | ! 141.793 us  |        }
   0)   user-29695   |   0.499 us    |        idle_cpu();
   0)   user-29695   |   0.528 us    |        rcu_irq_exit();
   0)   user-29695   | ! 146.663 us  |      }
   0)   user-29695   | * 15319.47 us |    }
   0)   user-29695   |   <========== |
   0)   user-29695   | * 15409.50 us |  }
   0)   user-29695   |               |  vpoll_ioctl [vpoll]() {
   0)   user-29695   |   0.982 us    |    _raw_spin_lock_irq();
   0)   user-29695   |               |    __wake_up_locked_key() {
   0)   user-29695   |               |      __wake_up_common() {
   0)   user-29695   |               |        ep_poll_callback() {
   0)   user-29695   |   0.963 us    |          _raw_read_lock_irqsave();
   0)   user-29695   |               |          __wake_up() {
   0)   user-29695   |               |            __wake_up_common_lock() {
   0)   user-29695   |   0.313 us    |              _raw_spin_lock_irqsave();
   0)   user-29695   |               |              __wake_up_common() {
   0)   user-29695   |               |                autoremove_wake_function() {
   0)   user-29695   |               |                  default_wake_function() {
   0)   user-29695   |               |                    try_to_wake_up() {
   0)   user-29695   |   0.547 us    |                      _raw_spin_lock_irqsave();
   0)   user-29695   |               |                      select_task_rq_fair() {
   0)   user-29695   |   0.338 us    |                        available_idle_cpu();
   0)   user-29695   |   1.135 us    |                        update_cfs_rq_h_load();
   0)   user-29695   |               |                        select_idle_sibling() {
   0)   user-29695   |   0.380 us    |                          available_idle_cpu();
   0)   user-29695   |   1.025 us    |                        }
   0)   user-29695   |   4.217 us    |                      }
   0)   user-29695   |               |                      native_smp_send_reschedule() {
   0)   user-29695   | + 28.904 us   |                        x2apic_send_IPI();
   0)   user-29695   | + 30.708 us   |                      }
   0)   user-29695   |   0.592 us    |                      _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 38.730 us   |                    }
   0)   user-29695   | + 39.400 us   |                  }
   0)   user-29695   | + 40.294 us   |                }
   0)   user-29695   | + 41.171 us   |              }
   0)   user-29695   |   0.355 us    |              _raw_spin_unlock_irqrestore();
   0)   user-29695   | + 43.087 us   |            }
   0)   user-29695   | + 43.691 us   |          }
   0)   user-29695   |   0.343 us    |          _raw_read_unlock_irqrestore();
   0)   user-29695   | + 46.535 us   |        }
   0)   user-29695   | + 47.886 us   |      }
   0)   user-29695   | + 48.642 us   |    }
   0)   user-29695   | + 53.748 us   |  }
   1)   user-29694   |               |  vpoll_ioctl [vpoll]() {
   1)   user-29694   |   0.664 us    |    _raw_spin_lock_irq();
   1)   user-29694   |               |    __wake_up_locked_key() {
   1)   user-29694   |               |      __wake_up_common() {
   1)   user-29694   |               |        ep_poll_callback() {
   1)   user-29694   |   0.353 us    |          _raw_read_lock_irqsave();
   1)   user-29694   |   0.340 us    |          _raw_read_unlock_irqrestore();
   1)   user-29694   |   1.777 us    |        }
   1)   user-29694   |   3.071 us    |      }
   1)   user-29694   |   3.734 us    |    }
   1)   user-29694   |   7.281 us    |  }
root@blue76815-virtual-machine:/sys/kernel/debug/tracing#
```
![](https://i.imgur.com/s4BMZ6r.png)

### 範例3.分析 vpoll_open 

```
step 1. 清除 current_tracer
$ echo nop > current_tracer

step 2. 設置要顯示調用棧的函數
$ echo vpoll_open  > set_graph_function

step 3. 配置跟蹤選項，開啟函式呼叫跟蹤，並跟蹤調用進程
$ echo function_graph > current_tracer
$ echo funcgraph-proc > trace_options

step 4. 開啟追蹤
$ echo 1 > tracing_on

step 5. 執行測試檔 ./user，再關閉追蹤：
$ /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
$ echo 0 > tracing_on

step 6.查看追蹤結果
$ cat trace
```
完整記錄

```
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo nop > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo vpoll_open >set_graph_function
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo function_graph > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo funcgraph-proc > trace_options
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 1 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
timeout...
GOT event 1
GOT event 1
GOT event 3
GOT event 2
GOT event 4
GOT event 10
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 0 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# cat trace
# tracer: function_graph
#
# CPU  TASK/PID         DURATION                  FUNCTION CALLS
# |     |    |           |   |                     |   |   |   |
   1)   user-29856   |               |  vpoll_open [vpoll]() {
   1)   user-29856   |               |    kmem_cache_alloc_trace() {
   1)   user-29856   |               |      _cond_resched() {
   1)   user-29856   |   0.212 us    |        rcu_all_qs();
   1)   user-29856   |   0.787 us    |      }
   1)   user-29856   |   0.192 us    |      should_failslab();
   1)   user-29856   |   1.856 us    |    }
   1)   user-29856   |   0.188 us    |    __init_waitqueue_head();
   1)   user-29856   |   4.929 us    |  }
root@blue76815-virtual-machine:/sys/kernel/debug/tracing#
``` 
![](https://i.imgur.com/0Bj7iUJ.png)

### 範例4.分析 vpoll_release

```
# 1. 清除 current_tracer
$ echo nop > current_tracer


# 2. 設置要顯示調用棧的函數
$ echo vpoll_release> set_graph_function

# 3. 配置跟蹤選項，開啟函式呼叫跟蹤，並跟蹤調用進程
$ echo function_graph > current_tracer
$ echo funcgraph-proc > trace_options

# 4. 開啟追蹤
$ echo 1 > tracing_on

# 5. 執行測試檔 ./user，再關閉追蹤：
$ /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
$ echo 0 > tracing_on

# 6.查看追蹤結果
$ cat trace
```
完整記錄

```
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo nop > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo vpoll_release> set_graph_function
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo function_graph > current_tracer
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo funcgraph-proc > trace_options
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 1 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# /home/blue76815/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1/./user
GOT event 1
GOT event 1
GOT event 3
timeout...
GOT event 2
GOT event 4
GOT event 10
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# echo 0 > tracing_on
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# cat trace
# tracer: function_graph
#
# CPU  TASK/PID         DURATION                  FUNCTION CALLS
# |     |    |           |   |                     |   |   |   |
   1)   user-30138   |               |  vpoll_release [vpoll]() {
   1)   user-30138   |               |    kfree() {
   1)   user-30138   |   0.558 us    |      __slab_free();
   1)   user-30138   |   2.942 us    |    }
   1)   user-30138   |   7.053 us    |  }
root@blue76815-virtual-machine:/sys/kernel/debug/tracing# 
```
![](https://i.imgur.com/OCY6nWT.png)



---

## 進階分析方法

### 方法1.trace-cmd
在 [2.1 ftrace 的原理与使用](https://hotttao.github.io/2020/01/03/linux_perf/03_ftrace/)提到
> ## 6. trace-cmd
> trace-cmd 可以把上面这些步骤给包装起来，通过同一个命令行工具，就可完成上述所有过程。下面是示例2 中跟踪 do_sys_open 的 trace-cmd 版本。
> 
> ```
> # 1. 安装
> $ sudo apt-get install trace-cmd
> # 2. 启动追踪
> $ trace-cmd record -p function_graph -g do_sys_open -O funcgraph-proc ls
> # 3. 查看追踪结果
> $ trace-cmd report
> ...
>               ls-12418 [000] 85558.075341: funcgraph_entry:                   |  do_sys_open() {
>               ls-12418 [000] 85558.075363: funcgraph_entry:                   |    getname() {
>               ls-12418 [000] 85558.075364: funcgraph_entry:                   |      getname_flags() {
>               ls-12418 [000] 85558.075364: funcgraph_entry:                   |        kmem_cache_alloc() {
>               ls-12418 [000] 85558.075365: funcgraph_entry:                   |          _cond_resched() {
>               ls-12418 [000] 85558.075365: funcgraph_entry:        0.074 us   |            rcu_all_qs();
>               ls-12418 [000] 85558.075366: funcgraph_exit:         1.143 us   |          }
>               ls-12418 [000] 85558.075366: funcgraph_entry:        0.064 us   |          should_failslab();
>               ls-12418 [000] 85558.075367: funcgraph_entry:        0.075 us   |          prefetch_freepointer();
>               ls-12418 [000] 85558.075368: funcgraph_entry:        0.085 us   |          memcg_kmem_put_cache();
>               ls-12418 [000] 85558.075369: funcgraph_exit:         4.447 us   |        }
>               ls-12418 [000] 85558.075369: funcgraph_entry:                   |        __check_object_size() {
>               ls-12418 [000] 85558.075370: funcgraph_entry:        0.132 us   |          __virt_addr_valid();
>               ls-12418 [000] 85558.075370: funcgraph_entry:        0.093 us   |          __check_heap_object();
>               ls-12418 [000] 85558.075371: funcgraph_entry:        0.059 us   |          check_stack_object();
>               ls-12418 [000] 85558.075372: funcgraph_exit:         2.323 us   |        }
>               ls-12418 [000] 85558.075372: funcgraph_exit:         8.411 us   |      }
>               ls-12418 [000] 85558.075373: funcgraph_exit:         9.195 us   |    }
> ...
> ```



---

```
blue76815@blue76815-virtual-machine:~/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1$ sudo trace-cmd record -p function_graph -O funcgraph-proc ./user
  plugin 'function_graph'
timeout...
GOT event 1
timeout...
GOT event 1
timeout...
GOT event 3
timeout...
GOT event 2
timeout...
GOT event 4
timeout...
GOT event 10
CPU0 data recorded at offset=0x561000
    0 bytes in size
CPU1 data recorded at offset=0x561000
    4096 bytes in size
```


![](https://i.imgur.com/LB1rByU.png)
跑完結果用
`sudo trace-cmd report >test` 將報告轉成 test2.txt檔


```
//用 -l 只能追蹤函式呼叫表層
sudo trace-cmd record -p function_graph -O funcgraph-proc -l vpoll_open -l vpoll_ioctl -l vpoll_poll -l vpoll_devnode ./user

sudo trace-cmd report >test2
```
test2 報表如下，可見用 -l 只能追蹤指定的函式呼叫表層
無法進一步分析指定函式內的 kernel 執行過程
```
CPU 0 is empty
cpus=2
            user-30538 [001] 98751.849976: funcgraph_entry:        6.907 us   |  vpoll_open();
            user-30538 [001] 98751.849991: funcgraph_entry:        1.820 us   |  vpoll_poll();
            user-30539 [001] 98752.850921: funcgraph_entry:      + 14.355 us  |  vpoll_ioctl();
            user-30538 [001] 98752.850942: funcgraph_entry:        1.228 us   |  vpoll_poll();
            user-30538 [001] 98752.850964: funcgraph_entry:        1.259 us   |  vpoll_ioctl();
            user-30538 [001] 98752.850968: funcgraph_entry:        0.543 us   |  vpoll_poll();
            user-30539 [001] 98753.851567: funcgraph_entry:        5.659 us   |  vpoll_ioctl();
            user-30538 [001] 98753.851574: funcgraph_entry:        0.541 us   |  vpoll_poll();
            user-30538 [001] 98753.851579: funcgraph_entry:        0.335 us   |  vpoll_ioctl();
            user-30538 [001] 98753.851581: funcgraph_entry:        0.160 us   |  vpoll_poll();
            user-30539 [001] 98754.852101: funcgraph_entry:      + 10.686 us  |  vpoll_ioctl();
            user-30538 [001] 98754.852117: funcgraph_entry:        1.268 us   |  vpoll_poll();
            user-30538 [001] 98754.852126: funcgraph_entry:        0.563 us   |  vpoll_ioctl();
            user-30538 [001] 98754.852128: funcgraph_entry:        0.254 us   |  vpoll_poll();
            user-30539 [001] 98755.852401: funcgraph_entry:        5.160 us   |  vpoll_ioctl();
            user-30538 [001] 98755.852407: funcgraph_entry:        0.453 us   |  vpoll_poll();
            user-30538 [001] 98755.852412: funcgraph_entry:        0.327 us   |  vpoll_ioctl();
            user-30538 [001] 98755.852413: funcgraph_entry:        0.151 us   |  vpoll_poll();
            user-30539 [001] 98756.853484: funcgraph_entry:        6.058 us   |  vpoll_ioctl();
            user-30538 [001] 98756.853492: funcgraph_entry:        0.534 us   |  vpoll_poll();
            user-30538 [001] 98756.853497: funcgraph_entry:        0.341 us   |  vpoll_ioctl();
            user-30538 [001] 98756.853498: funcgraph_entry:        0.170 us   |  vpoll_poll();
            user-30539 [001] 98757.854314: funcgraph_entry:        6.330 us   |  vpoll_ioctl();
            user-30538 [001] 98757.854323: funcgraph_entry:        0.542 us   |  vpoll_poll();
            user-30538 [001] 98757.854329: funcgraph_entry:        0.416 us   |  vpoll_ioctl();
```


---

[ftrace和trace-cmd：跟踪内核函数的利器](https://blog.csdn.net/weixin_44410537/article/details/103587609)
### trace-cmd 最終解法( -g 同時追蹤多個指定函數的執行過程)
終端機輸入
`trace-cmd record -h` 可查到 trace-cmd 的使用說明
每個追蹤函數 -g 代表 set graph function

參考 [2.1 ftrace 的原理与使用 6. trace-cmd](https://hotttao.github.io/2020/01/03/linux_perf/03_ftrace/) 的用法 

輸入
```
sudo trace-cmd record -p function_graph -O funcgraph-proc -g vpoll_open -g vpoll_release -g vpoll_ioctl -g vpoll_poll -g vpoll_devnode ./user
```

能同時追蹤 `vpoll_open`，`vpoll_release`， `vpoll_ioctl`，`vpoll_poll`，`vpoll_devnode`這幾個註冊到 file_operations fops 內，在 kernel 的執行狀況

```
blue76815@blue76815-virtual-machine:~/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1$ sudo trace-cmd record -p function_graph -O funcgraph-proc -g vpoll_open -g vpoll_release -g vpoll_ioctl -g vpoll_poll -g vpoll_devnode ./user
  plugin 'function_graph'
timeout...
GOT event 1
GOT event 1
timeout...
GOT event 3
timeout...
GOT event 2
timeout...
GOT event 4
timeout...
GOT event 10
CPU0 data recorded at offset=0x561000
    0 bytes in size
CPU1 data recorded at offset=0x561000
    36864 bytes in size
blue76815@blue76815-virtual-machine:~/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1$ sudo trace-cmd report >test3    
```


跑完結果用
`sudo trace-cmd report >test3`
![](https://i.imgur.com/PTOhYf7.png)
節錄 test3 部分結果

```
CPU 0 is empty
cpus=2
            user-30740 [001] 100222.951566: funcgraph_entry:                   |  vpoll_open() {
            user-30740 [001] 100222.951568: funcgraph_entry:                   |    kmem_cache_alloc_trace() {
            user-30740 [001] 100222.951568: funcgraph_entry:                   |      _cond_resched() {
            user-30740 [001] 100222.951568: funcgraph_entry:        0.209 us   |        rcu_all_qs();
            user-30740 [001] 100222.951569: funcgraph_exit:         0.697 us   |      }
            user-30740 [001] 100222.951569: funcgraph_entry:        0.166 us   |      should_failslab();
            user-30740 [001] 100222.951569: funcgraph_exit:         1.663 us   |    }
            user-30740 [001] 100222.951570: funcgraph_entry:        0.166 us   |    __init_waitqueue_head();
            user-30740 [001] 100222.951570: funcgraph_exit:         4.163 us   |  }
            user-30740 [001] 100222.951581: funcgraph_entry:                   |  vpoll_poll() {
            user-30740 [001] 100222.951582: funcgraph_entry:                   |    ep_ptable_queue_proc() {
            user-30740 [001] 100222.951582: funcgraph_entry:                   |      kmem_cache_alloc() {
            user-30740 [001] 100222.951582: funcgraph_entry:                   |        _cond_resched() {
            user-30740 [001] 100222.951582: funcgraph_entry:        0.181 us   |          rcu_all_qs();
            user-30740 [001] 100222.951583: funcgraph_exit:         0.520 us   |        }
            user-30740 [001] 100222.951583: funcgraph_entry:        0.162 us   |        should_failslab();
            user-30740 [001] 100222.951584: funcgraph_exit:         1.684 us   |      }
            user-30740 [001] 100222.951584: funcgraph_entry:                   |      add_wait_queue() {
            user-30740 [001] 100222.951584: funcgraph_entry:        0.165 us   |        _raw_spin_lock_irqsave();
            user-30740 [001] 100222.951584: funcgraph_entry:        0.172 us   |        __lock_text_start();
            user-30740 [001] 100222.951585: funcgraph_exit:         0.820 us   |      }
            user-30740 [001] 100222.951585: funcgraph_exit:         3.037 us   |    }
            user-30740 [001] 100222.951585: funcgraph_exit:         3.571 us   |  }
            user-30741 [001] 100223.953007: funcgraph_entry:                   |  vpoll_ioctl() {
            user-30741 [001] 100223.953009: funcgraph_entry:        0.615 us   |    _raw_spin_lock_irq();
            user-30741 [001] 100223.953010: funcgraph_entry:                   |    __wake_up_locked_key() {
            user-30741 [001] 100223.953010: funcgraph_entry:                   |      __wake_up_common() {
            user-30741 [001] 100223.953011: funcgraph_entry:                   |        ep_poll_callback() {
            user-30741 [001] 100223.953012: funcgraph_entry:        0.770 us   |          _raw_read_lock_irqsave();
            user-30741 [001] 100223.953012: funcgraph_entry:                   |          __wake_up() {
            user-30741 [001] 100223.953013: funcgraph_entry:                   |            __wake_up_common_lock() {
            user-30741 [001] 100223.953013: funcgraph_entry:        0.294 us   |              _raw_spin_lock_irqsave();
            user-30741 [001] 100223.953014: funcgraph_entry:                   |              __wake_up_common() {
            user-30741 [001] 100223.953014: funcgraph_entry:                   |                autoremove_wake_function() {
            user-30741 [001] 100223.953014: funcgraph_entry:                   |                  default_wake_function() {
            user-30741 [001] 100223.953015: funcgraph_entry:                   |                    try_to_wake_up() {
            user-30741 [001] 100223.953015: funcgraph_entry:        0.310 us   |                      _raw_spin_lock_irqsave();
            user-30741 [001] 100223.953016: funcgraph_entry:                   |                      select_task_rq_fair() {
            user-30741 [001] 100223.953016: funcgraph_entry:                   |                        select_idle_sibling() {
            user-30741 [001] 100223.953016: funcgraph_entry:        0.314 us   |                          available_idle_cpu();
            user-30741 [001] 100223.953017: funcgraph_entry:        0.350 us   |                          cpus_share_cache();
            user-30741 [001] 100223.953018: funcgraph_exit:         1.536 us   |                        }
            user-30741 [001] 100223.953018: funcgraph_exit:         2.188 us   |                      }
            user-30741 [001] 100223.953018: funcgraph_entry:        0.297 us   |                      _raw_spin_lock();
            user-30741 [001] 100223.953019: funcgraph_entry:                   |                      update_rq_clock() {
            user-30741 [001] 100223.953019: funcgraph_entry:        0.324 us   |                        update_rq_clock.part.82();
            user-30741 [001] 100223.953020: funcgraph_exit:         0.916 us   |                      }
            user-30741 [001] 100223.953020: funcgraph_entry:                   |                      ttwu_do_activate() {
            user-30741 [001] 100223.953020: funcgraph_entry:                   |                        activate_task() {
            user-30741 [001] 100223.953021: funcgraph_entry:                   |                          psi_task_change() {
            user-30741 [001] 100223.953021: funcgraph_entry:        0.582 us   |                            record_times();
            user-30741 [001] 100223.953022: funcgraph_entry:        0.436 us   |                            record_times();
            user-30741 [001] 100223.953023: funcgraph_entry:        0.368 us   |                            record_times();
            user-30741 [001] 100223.953024: funcgraph_entry:        0.383 us   |                            record_times();
            user-30741 [001] 100223.953024: funcgraph_entry:        0.329 us   |                            record_times();
            user-30741 [001] 100223.953025: funcgraph_exit:         4.200 us   |                          }
            user-30741 [001] 100223.953025: funcgraph_entry:                   |                          enqueue_task_fair() {
            user-30741 [001] 100223.953026: funcgraph_entry:                   |                            enqueue_entity() {
            user-30741 [001] 100223.953026: funcgraph_entry:                   |                              update_curr() {
            user-30741 [001] 100223.953026: funcgraph_entry:        0.323 us   |                                update_min_vruntime();
            user-30741 [001] 100223.953027: funcgraph_entry:        0.313 us   |                                cpuacct_charge();
            user-30741 [001] 100223.953027: funcgraph_entry:                   |                                __cgroup_account_cputime() {
            user-30741 [001] 100223.953028: funcgraph_entry:        0.318 us   |                                  cgroup_rstat_updated();
            user-30741 [001] 100223.953028: funcgraph_exit:         0.875 us   |                                }
            user-30741 [001] 100223.953029: funcgraph_exit:         2.915 us   |                              }
            user-30741 [001] 100223.953029: funcgraph_entry:        0.339 us   |                              __update_load_avg_se();
            user-30741 [001] 100223.953030: funcgraph_entry:        0.356 us   |                              __update_load_avg_cfs_rq();
            user-30741 [001] 100223.953030: funcgraph_entry:        0.280 us   |                              update_cfs_group();
            user-30741 [001] 100223.953031: funcgraph_entry:        0.339 us   |                              account_entity_enqueue();
            user-30741 [001] 100223.953032: funcgraph_entry:        0.371 us   |                              __enqueue_entity();
            user-30741 [001] 100223.953032: funcgraph_exit:         6.793 us   |                            }
            user-30741 [001] 100223.953033: funcgraph_entry:        0.324 us   |                            __update_load_avg_se();
            user-30741 [001] 100223.953033: funcgraph_entry:        0.289 us   |                            __update_load_avg_cfs_rq();
            user-30741 [001] 100223.953034: funcgraph_entry:                   |                            update_cfs_group() {
            user-30741 [001] 100223.953034: funcgraph_entry:                   |                              reweight_entity() {
            user-30741 [001] 100223.953035: funcgraph_entry:                   |                                update_curr() {
            user-30741 [001] 100223.953035: funcgraph_entry:        0.281 us   |                                  update_min_vruntime();
            user-30741 [001] 100223.953035: funcgraph_exit:         0.863 us   |                                }
            user-30741 [001] 100223.953036: funcgraph_entry:        0.276 us   |                                account_entity_dequeue();
            user-30741 [001] 100223.953036: funcgraph_entry:        0.298 us   |                                account_entity_enqueue();
            user-30741 [001] 100223.953037: funcgraph_exit:         2.637 us   |                              }
            user-30741 [001] 100223.953037: funcgraph_exit:         3.338 us   |                            }
            user-30741 [001] 100223.953038: funcgraph_entry:        0.286 us   |                            hrtick_update();
            user-30741 [001] 100223.953039: funcgraph_exit:       + 12.972 us  |                          }
            user-30741 [001] 100223.953044: funcgraph_exit:       + 23.540 us  |                        }
            user-30741 [001] 100223.953044: funcgraph_entry:                   |                        ttwu_do_wakeup() {
            user-30741 [001] 100223.953045: funcgraph_entry:                   |                          check_preempt_curr() {
            user-30741 [001] 100223.953045: funcgraph_entry:                   |                            check_preempt_wakeup() {
            user-30741 [001] 100223.953045: funcgraph_entry:        0.314 us   |                              update_curr();
            user-30741 [001] 100223.953046: funcgraph_entry:        0.344 us   |                              wakeup_preempt_entity.isra.77();
            user-30741 [001] 100223.953047: funcgraph_exit:         1.563 us   |                            }
            user-30741 [001] 100223.953047: funcgraph_exit:         2.233 us   |                          }
            user-30741 [001] 100223.953049: funcgraph_exit:         4.315 us   |                        }
            user-30741 [001] 100223.953049: funcgraph_exit:       + 28.771 us  |                      }
            user-30741 [001] 100223.953049: funcgraph_entry:        0.301 us   |                      __lock_text_start();
            user-30741 [001] 100223.953050: funcgraph_exit:       + 35.114 us  |                    }
            user-30741 [001] 100223.953050: funcgraph_exit:       + 35.743 us  |                  }
            user-30741 [001] 100223.953050: funcgraph_exit:       + 36.381 us  |                }
            user-30741 [001] 100223.953051: funcgraph_exit:       + 37.161 us  |              }
            user-30741 [001] 100223.953051: funcgraph_entry:        0.292 us   |              __lock_text_start();
            user-30741 [001] 100223.953052: funcgraph_exit:       + 38.919 us  |            }
            user-30741 [001] 100223.953052: funcgraph_exit:       + 39.479 us  |          }
            user-30741 [001] 100223.953052: funcgraph_entry:        0.324 us   |          _raw_read_unlock_irqrestore();
            user-30741 [001] 100223.953053: funcgraph_exit:       + 41.994 us  |        }
            user-30741 [001] 100223.953053: funcgraph_exit:       + 43.290 us  |      }
            user-30741 [001] 100223.953054: funcgraph_exit:       + 43.882 us  |    }
            user-30741 [001] 100223.953056: funcgraph_entry:                   |    smp_irq_work_interrupt() {
            user-30741 [001] 100223.953056: funcgraph_entry:                   |      irq_enter() {
            user-30741 [001] 100223.953056: funcgraph_entry:        0.317 us   |        rcu_irq_enter();
            user-30741 [001] 100223.953057: funcgraph_exit:         0.906 us   |      }
            user-30741 [001] 100223.953059: funcgraph_entry:                   |      __wake_up() {
            user-30741 [001] 100223.953059: funcgraph_entry:                   |        __wake_up_common_lock() {
            user-30741 [001] 100223.953059: funcgraph_entry:        0.288 us   |          _raw_spin_lock_irqsave();
            user-30741 [001] 100223.953060: funcgraph_entry:        0.323 us   |          __wake_up_common();
            user-30741 [001] 100223.953061: funcgraph_entry:        0.293 us   |          __lock_text_start();
            user-30741 [001] 100223.953061: funcgraph_exit:         2.054 us   |        }
            user-30741 [001] 100223.953061: funcgraph_exit:         2.677 us   |      }
            user-30741 [001] 100223.953062: funcgraph_entry:                   |      __wake_up() {
            user-30741 [001] 100223.953062: funcgraph_entry:                   |        __wake_up_common_lock() {
            user-30741 [001] 100223.953062: funcgraph_entry:        0.291 us   |          _raw_spin_lock_irqsave();
            user-30741 [001] 100223.953063: funcgraph_entry:                   |          __wake_up_common() {
            user-30741 [001] 100223.953064: funcgraph_entry:                   |            autoremove_wake_function() {
            user-30741 [001] 100223.953064: funcgraph_entry:                   |              default_wake_function() {
            user-30741 [001] 100223.953064: funcgraph_entry:                   |                try_to_wake_up() {
            user-30741 [001] 100223.953064: funcgraph_entry:        0.777 us   |                  _raw_spin_lock_irqsave();
            user-30741 [001] 100223.953066: funcgraph_entry:                   |                  select_task_rq_fair() {
            user-30741 [001] 100223.953066: funcgraph_entry:                   |                    select_idle_sibling() {
            user-30741 [001] 100223.953066: funcgraph_entry:        0.309 us   |                      available_idle_cpu();
            user-30741 [001] 100223.953067: funcgraph_exit:         0.952 us   |                    }
            user-30741 [001] 100223.953067: funcgraph_exit:         1.592 us   |                  }
            user-30741 [001] 100223.953068: funcgraph_entry:        0.289 us   |                  _raw_spin_lock();
            user-30741 [001] 100223.953068: funcgraph_entry:                   |                  update_rq_clock() {
            user-30741 [001] 100223.953068: funcgraph_entry:        0.309 us   |                    update_rq_clock.part.82();
            user-30741 [001] 100223.953069: funcgraph_exit:         0.875 us   |                  }
            user-30741 [001] 100223.953069: funcgraph_entry:                   |                  ttwu_do_activate() {
            user-30741 [001] 100223.953070: funcgraph_entry:                   |                    activate_task() {
            user-30741 [001] 100223.953070: funcgraph_entry:                   |                      psi_task_change() {
            user-30741 [001] 100223.953070: funcgraph_entry:        0.445 us   |                        record_times();
            user-30741 [001] 100223.953071: funcgraph_entry:        0.312 us   |                        record_times();
            user-30741 [001] 100223.953072: funcgraph_entry:        0.313 us   |                        record_times();
            user-30741 [001] 100223.953072: funcgraph_entry:        0.332 us   |                        record_times();
            user-30741 [001] 100223.953073: funcgraph_entry:        0.314 us   |                        record_times();
            user-30741 [001] 100223.953074: funcgraph_exit:         3.813 us   |                      }
            user-30741 [001] 100223.953074: funcgraph_entry:                   |                      enqueue_task_fair() {
            user-30741 [001] 100223.953074: funcgraph_entry:                   |                        enqueue_entity() {
            user-30741 [001] 100223.953075: funcgraph_entry:                   |                          update_curr() {
            user-30741 [001] 100223.953075: funcgraph_entry:        0.331 us   |                            update_min_vruntime();
            user-30741 [001] 100223.953076: funcgraph_entry:        0.310 us   |                            cpuacct_charge();
            user-30741 [001] 100223.953076: funcgraph_entry:                   |                            __cgroup_account_cputime() {
            user-30741 [001] 100223.953076: funcgraph_entry:        0.284 us   |                              cgroup_rstat_updated();
            user-30741 [001] 100223.953077: funcgraph_exit:         0.855 us   |                            }
            user-30741 [001] 100223.953077: funcgraph_exit:         2.716 us   |                          }
            user-30741 [001] 100223.953078: funcgraph_entry:                   |                          __update_load_avg_se() {
            user-30741 [001] 100223.953078: funcgraph_entry:        0.287 us   |                            __accumulate_pelt_segments();
            user-30741 [001] 100223.953079: funcgraph_exit:         1.040 us   |                          }
            user-30741 [001] 100223.953079: funcgraph_entry:        0.289 us   |                          __update_load_avg_cfs_rq();
            user-30741 [001] 100223.953080: funcgraph_entry:        0.285 us   |                          update_cfs_group();
            user-30741 [001] 100223.953080: funcgraph_entry:        0.372 us   |                          account_entity_enqueue();
            user-30741 [001] 100223.953081: funcgraph_entry:        0.311 us   |                          __enqueue_entity();
            user-30741 [001] 100223.953081: funcgraph_exit:         7.109 us   |                        }
            user-30741 [001] 100223.953082: funcgraph_entry:        0.328 us   |                        __update_load_avg_se();
            user-30741 [001] 100223.953082: funcgraph_entry:        0.289 us   |                        __update_load_avg_cfs_rq();
            user-30741 [001] 100223.953083: funcgraph_entry:                   |                        update_cfs_group() {
            user-30741 [001] 100223.953083: funcgraph_entry:                   |                          reweight_entity() {
            user-30741 [001] 100223.953084: funcgraph_entry:                   |                            update_curr() {
            user-30741 [001] 100223.953084: funcgraph_entry:        0.317 us   |                              update_min_vruntime();
            user-30741 [001] 100223.953085: funcgraph_exit:         0.894 us   |                            }
            user-30741 [001] 100223.953085: funcgraph_entry:        0.331 us   |                            account_entity_dequeue();
            user-30741 [001] 100223.953086: funcgraph_entry:        0.301 us   |                            account_entity_enqueue();
            user-30741 [001] 100223.953086: funcgraph_exit:         3.271 us   |                          }
            user-30741 [001] 100223.953087: funcgraph_exit:         3.901 us   |                        }
            user-30741 [001] 100223.953087: funcgraph_entry:        0.278 us   |                        hrtick_update();
            user-30741 [001] 100223.953088: funcgraph_exit:       + 13.690 us  |                      }
            user-30741 [001] 100223.953088: funcgraph_exit:       + 18.425 us  |                    }
            user-30741 [001] 100223.953088: funcgraph_entry:                   |                    ttwu_do_wakeup() {
            user-30741 [001] 100223.953089: funcgraph_entry:                   |                      check_preempt_curr() {
            user-30741 [001] 100223.953089: funcgraph_entry:                   |                        check_preempt_wakeup() {
            user-30741 [001] 100223.953089: funcgraph_entry:        0.306 us   |                          update_curr();
            user-30741 [001] 100223.953090: funcgraph_entry:        0.281 us   |                          wakeup_preempt_entity.isra.77();
            user-30741 [001] 100223.953090: funcgraph_entry:        0.378 us   |                          set_next_buddy();
            user-30741 [001] 100223.953091: funcgraph_entry:        0.305 us   |                          resched_curr();
            user-30741 [001] 100223.953092: funcgraph_exit:         2.794 us   |                        }
            user-30741 [001] 100223.953092: funcgraph_exit:         3.435 us   |                      }
            user-30741 [001] 100223.953092: funcgraph_exit:         4.208 us   |                    }
            user-30741 [001] 100223.953093: funcgraph_exit:       + 23.466 us  |                  }
            user-30741 [001] 100223.953093: funcgraph_entry:        0.305 us   |                  __lock_text_start();
            user-30741 [001] 100223.953094: funcgraph_exit:       + 29.517 us  |                }
            user-30741 [001] 100223.953094: funcgraph_exit:       + 30.065 us  |              }
            user-30741 [001] 100223.953094: funcgraph_exit:       + 30.639 us  |            }
            user-30741 [001] 100223.953094: funcgraph_exit:       + 31.581 us  |          }
            user-30741 [001] 100223.953095: funcgraph_entry:        0.290 us   |          __lock_text_start();
            user-30741 [001] 100223.953095: funcgraph_exit:       + 33.319 us  |        }
            user-30741 [001] 100223.953096: funcgraph_exit:       + 33.895 us  |      }
            user-30741 [001] 100223.953096: funcgraph_entry:                   |      irq_exit() {
            user-30741 [001] 100223.953096: funcgraph_entry:        0.292 us   |        idle_cpu();
            user-30741 [001] 100223.953097: funcgraph_entry:        0.302 us   |        rcu_irq_exit();
            user-30741 [001] 100223.953097: funcgraph_exit:         1.451 us   |      }
            user-30741 [001] 100223.953098: funcgraph_exit:       + 42.067 us  |    }
            user-30741 [001] 100223.953098: funcgraph_exit:       + 92.360 us  |  }
```
            
### 方法2.trace-cmd+kernelshark
[通过trace-cmd和kernelshark简化Ftrace的使用](https://blog.colorfulshark.net/2019/12/21/simplify-ftrace-with-tracecmd-and-kernelshark.html)

step1. 安裝 trace-cmd 和 kernelshark
```
sudo apt-get install trace-cmd kernelshark
```
step2.分析時一樣
先掛載模組
`sudo insmod vpoll.ko`

step3.`trace-cmd`指令分析在跑 `./user` 時的 system call過程
```
//用 trace-cmd 指令，產生 trace.dat分析報表
sudo trace-cmd record -p function_graph -O funcgraph-proc -g vpoll_open -g vpoll_release -g vpoll_ioctl -g vpoll_poll -g vpoll_devnode ./user

//用 kernelshark 軟體開啟 trace.dat分析報表
kernelshark trace.dat
```

完整執行過程

```
blue76815@blue76815-virtual-machine:~/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1$ sudo trace-cmd record -p function_graph -O funcgraph-proc -g vpoll_open -g vpoll_release -g vpoll_ioctl -g vpoll_poll -g vpoll_devnode ./user
  plugin 'function_graph'
timeout...
GOT event 1
timeout...
GOT event 1
timeout...
GOT event 3
timeout...
GOT event 2
timeout...
GOT event 4
timeout...
GOT event 10
CPU0 data recorded at offset=0x561000
    36864 bytes in size
CPU1 data recorded at offset=0x56a000
    0 bytes in size
blue76815@blue76815-virtual-machine:~/桌面/2021成大暑期/quiz4_vpoll/8月25日進度/quiz4-1-type1$ kernelshark trace.dat
Gtk-Message: 17:36:48.870: Failed to load module "canberra-gtk-module"
CPU 1 is empty
```

![](https://i.imgur.com/wXll5Tm.png)

用 kernelshark 軟體開啟 trace.dat 分析報表
效果如下
![](https://i.imgur.com/Gr7bvQb.png)
