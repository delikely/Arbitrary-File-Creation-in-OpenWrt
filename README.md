# Arbitrary-File-Creation-in-OpenWrt
## Arbitrary File Creation in OpenWrt

There is no restriction on the path of the system log file(`logread`),which lead to **overwritten important file**,such as /etc/passwd. if `/etc/passwd` was overwritened,it will cause a denial of service. when a router was infected, The only way to fix  is **Flash Firmware**.

In the background, function [`start_service_file`](https://github.com/openwrt/openwrt/blob/fd28ef59db92da245debf207892fad8e1a0d9e45/package/system/ubox/files/log.init#L44) will deal the system logging  request and use logread` to save log in specified file.

```sh
PROG=/sbin/logread  

start_service_file()                                                              
{                                                                                 
        PIDCOUNT="$(( ${PIDCOUNT} + 1))"                                          
        local pid_file="/var/run/logread.${PIDCOUNT}.pid"                         
                                                                                  
        [ "$2" = 0 ] || {                                                         
                echo "validation failed"                                          
                return 1                                                          
        }                                                                         
        [ -z "${log_file}" ] && return                                            
                                                                                  
        mkdir -p "$(dirname "${log_file}")"                                       
                                                                                  
        procd_open_instance                                                       
        procd_set_param command "$PROG" -f -F "$log_file" -p "$pid_file"          
        [ -n "${log_size}" ] && procd_append_param command -S "$log_size"         
        procd_close_instance                                                      
}   
```

### POC

#### 1. set the path of  sustem log  and buffer size

In System -> logging page,`System log buffer size`set to `1`KiB ,`Write system log to file` set to `/etc/passwd`. At the end , clicking  "Save & Apply" button.

![set value](images/image-20200110233312316.png)



#### 2. reboot or wait a moment

In order to produce some log to overwriten `/etc/passwd` , it need to reboot OpenWrt (the fasest way ) or wait a monent.

#### 3. result

 Denial of service: the LuCi web page display "Dad Gateway". the same time , the internet was offline etc.

![WEB DOS](images/image-20200110233616372.png)

