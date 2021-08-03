# Encryption

## Symetric Encryption (Message + Key)

1. I have a message "Hello world"
2. Encryp the message with a key "KEY" + "Hello world" = XHALSJHRLWES
3. Send the message encrypted. Send: XHALSJHRLWES + KEY


## Asymetric Encryption [SSH] (Message + Pub Key (Lock) + Priv Key)

1. I have a message "Hello world"
2. I have a "Public Key" and "Private Key". id_rsa.pem (pub) + id_rsa (priv)
3. Private key id_rsa have only the owner he doesn't send. id_rsa.pem (public) send to the client.
4. Encrypt the message with public key (id_rsa.pem) but only with the private key (id_rsa) the message will be de-encrypted.
5. The message encrypted it'll be sended to the client, the hacker have a possibility to capture but he doesn't have the possibility to decrypted.

```
ssh-keygen
id_rsa > priv only to the client
id_rsa.pub > pub > send to the server $ cat ~/.ssh/authorized_keys

client $ ssh -i id_rsa user@server
Succefully Logged In!
```

The problem is when we need to scale up in a number of servers, the copy of the public key are necessary in all the servers.


## Asymetric Encryption in the perspective of the Web Server

1. Server generate the public and private key.

```
openssl genrsa -out my-bank.key 1024
my-bank.key

openssl rsa -in my-bank.key -pubout > mybank.pem
my-bank.key mybank.pem
```

2. The hacker capture the pub key from the server mybank.pem

3. Server send the pub key (mybank.pem) to the client and the client (web browser) encrypt the symetric key (client.key) with the server public key (mybank.pem).

4. The hacker has a copy of symetric key encrypted with the server public key. The hacker has server public key (mybank.pub) and the message encrypted that contain symetric key. But the hacker hasn't the server priv key to decrypt the message and take the client symetric key.

5. The server receive the encrypted message with the cliente symetric key, the server has decrypted the message because has the server private key (my-bank.key) and he took the client symetric key to decrypt all the future messages.

6. The client send the message encrypted with te cliente symetric key that the server previusly took. The hacker capture the encrypted message but hasn't the symetric private key to decrypted.

## Certificate Authority

A hacker has a possibility to clone the web site (pishing). The server send the CERTIFICATE that previusly singed by the CERTIFICATE AUTHORITY. The CERTIFICATE show who is the entity and who sing the certificate.

1. The server create Certificate Signing Request (CSR)

```
$ openssl req -new -key my-bank.key -out my-bank.csr -subj "/C=US/ST=CA/O=MyOrg, Inc./CN=my-bank.com"
my-bank,key my-bank.csr
```

2. Send the CSR to CA. CA send the CERTIFICATE SIGNED (my-bank.crt). All the browsers has the public key of the all CA (Symantec, Global Sign, Digicert) but don't have the private key of all of them. 

3. Server send the CERTIFICATE SIGNED (my-bank.crt) to the client. The client has the CA public key (into the browser) and decrypted the server public key.

4. Client with the server public key encrypt the symetric key to send the server.

5. Server recive the encrypted client symetric key and decrypted.

## PKI (Public Key Infrastructure)

All of above detailed is called PKI.

## Important!!!

**Only encrypt the data with the public key and decrypt with the private key, it doesn't possible to decrypt the data with the same key that the encrypted.**

* Certificate (Public Key): *.crt or *.pem

```
server.crt
server.pem
client.crt
client.pem
```

* Private key: *.key or *-key.pem

```
server.key
server-key.pem
client.key
client-key.pem
```

