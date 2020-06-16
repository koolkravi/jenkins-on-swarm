sudo docker swarm init --advertise-addr 10.0.0.6
#docker swarm join-token worker
sudo docker swarm join --token SWMTKN-1-3kt8updl8dgxcfqbehn7ub192adqcxwslv5tgtkbhcyo5aiuis-05ljk1qrfn056ezfn3bw4eamu 10.0.0.5:2377

