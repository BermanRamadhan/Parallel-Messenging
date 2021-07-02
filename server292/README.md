# Server Parallel
Guide ini dibuat dengan signal-server 2.92


## Requirement
* JDK 11
* Certificate SSL domain
* Google Recaptcha
* Firebase Cloud Messaging (It used to be GCM)
* Twilio
* AWS 

1. Membuat File config.yml tersendiri, disimpan di <parallelprojectname>/service/config

2.	Build server dengan config buatan sekaligus mengcompile file jar
```
mvn clean install -DskipTests
```

3. Generate value untuk di field UnidentifiedDelivery

Generate menggunakan command dibawah ini (SIMPAN SEMUA CODENYA DI NOTEPAD) public key akan digunakan di android
```
java -jar service/target/TextSecureServer-2.92.jar certificate -ca
```

Menggunakan Private key untuk menggenerate keys certificate selanjutnya (key id bebas)
```
java -jar service/target/TextSecureServer-2.92.jar certificate --key <priv_key_from_step_above> --id <the_key_ID>
```

4.	Run postgres, redis, coturn ( Direkomendasikan untuk menggunakan docker-compose).

5.	Migrate databases
```
java -jar service/target/TextSecureServer-2.92.jar abusedb migrate service/config/config.yml
java -jar service/target/TextSecureServer-2.92.jar accountdb migrate service/config/config.yml
java -jar service/target/TextSecureServer-2.92.jar keysdb migrate service/config/config.yml
java -jar service/target/TextSecureServer-2.92.jar messagedb migrate service/config/config.yml
```

6.	menjalankan server (config.yml dari step pertama)
```
java -jar service/target/TextSecureServer-2.92.jar server service/config/config.yml
```

7. Menjalankan server di background (run continously), menggunakan nohup
```
nohup java -jar service/target/TextSecureServer-2.92.jar server service/config/config.yml > /dev/null &
```

## Konfigurasi Nginx & Generate SSL Certificate dengan Let's Encrypt

jika sudah memiliki SSL Certificate, bisa menggunakan [contoh nginx config](./example-nginx.conf) di `Step 4` dan skip `Step 6 - 9`.

1. install nginx
```
sudo apt install nginx     
```

2. Install Certbot untuk Let's Encrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get install python-certbot-nginx
```

3. Mengizinkan Nginx untuk diakses dari luar firewall
```
sudo ufw allow 'Nginx Full'
```

4. Membuat konfigurasi server `/etc/nginx/sites-enabled/domain.com`. 
```
server {
  listen 80;
  listen [::]:80;

  server_name domain.com;
}
```

5. Reload Nginx untuk menerapkan konfigutari baru
```
sudo nginx -s reload

```

6. Run certbot untuk generate SSL Certificate
```
certbot --nginx -d domain.com
``` 

7. Saat ditanya `Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.` direkomendasikan untuk memilih `2: Redirect`. setelah selesai file akan disimpan di :
```
etc
└── letsencrypt
    └── live 
        └── domain.com
            └── *.pem
```

8. Update nginx sesuai kebutuhan, bisa di lihat di [the example here](./example-nginx.conf).

9.  Check apabila nginx sudah sesuai
```
sudo nginx -t
```

10. Jika tidak ada error, lanjut reload kembali nginx untuk update konfigurasi
```
sudo nginx -s reload
```
