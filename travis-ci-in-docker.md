### Travis CI
https://travis-ci.com

```
sudo docker run -it -v $(pwd):/app ruby:2.3 sh
# /app is a random volume tha we are mounting to the docker container
gem install travis -v 1.8.10
travis
travis login --com
# log into github

travis encrypt-file service.json -r keentaussig/fibonacci-k8s --com
# created:  openssl aes-256-cbc -K $encrypted_0c35eebf403c_key -iv $encrypted_0c35eebf403c_iv -in service.json.enc -out service.json -d
```
