docker service create --name registry --publish published=5000,target=5000 registry:2

curl http://10.0.0.6:5000/v2

#from browser
http://127.0.0.1:5000/v2/