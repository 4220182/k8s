kubectl create secret tls tls-cert --cert=cert.pem --key=key.pem -n istio-system
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -subj '/CN=ambassador-cert' -nodes 


