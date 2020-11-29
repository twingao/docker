# Docker使用gzip压缩导出/导入镜像

导出镜像

    docker save <myimage>:<tag> | gzip > <myimage>.<tag>.tar.gz

举例

    docker save grafana/grafana:5.2.1 | gzip > grafana.5.2.1.tar.gz

导入镜像

    gunzip -c <myimage>.<tag>.tar.gz | docker load

举例

    gunzip -c grafana.5.2.1.tar.gz | docker load
