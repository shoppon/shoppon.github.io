kubectl cp <new package> openstack/coaster-all-0:tmp/ -c coaster-other 命令将对接包拷贝到coaster-other容器内/tmp目录下。

roller env --env 1 --product-info --avaliable --component solution_storage |grep pkg_name

使用kubectl exec -it -n openstack coaster-all-0 -c coaster-other bash 命令进入到coaster-other 容器内

cp /tmp/<new package> 1602728085.06.es 