make image-chaos-daemon
->
ghcr.io/chaos-mesh/chaos-daemon:latest
tag it:
docker tag ghcr.io/chaos-mesh/chaos-daemon:latest 192.168.1.6:5000/chaos-daemon
# docker push 192.168.1.6:5000/chaos-daemon

kubectl create ns chaos-mesh

helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock --version 2.7.0 --set chaosDaemon.image.registry=192.168.1.6:5000 --set chaosDaemon.image.repository=chaos-daemon --set chaosDaemon.image.tag=latest --set dashboard.securityMode=false --set chaosDaemon.imagePullPolicy=Always

helm upgrade chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock --version 2.7.0 --set chaosDaemon.image.registry=192.168.1.6:5000 --set chaosDaemon.image.repository=chaos-daemon --set chaosDaemon.image.tag=latest --set dashboard.securityMode=false --set chaosDaemon.imagePullPolicy=Always

# ssh tunnel: 
ssh -C -N -L local-port:Node-IP:Service-Node-Port user@ip -i cert_file