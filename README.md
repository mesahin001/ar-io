# ar-io
Ar.io node kurulumu


# Güncelleme ve gerekli paketler
sudo apt update -y && sudo apt upgrade -y && sudo apt install -y curl openssh-server docker-compose git certbot nginx sqlite3 build-essential && sudo systemctl enable ssh && curl -sSL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add - && echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list && sudo apt-get update -y && sudo apt-get install -y yarn && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash && source ~/.bashrc

# Contabo veya Hetzner calistiriyorsaniz bunlari kullanmaniza gerek yok, orda portlar zaten acik.
sudo ufw enable

sudo ufw allow 22

sudo ufw allow 80

sudo ufw allow 443


# ngix ve docker
sudo apt install nginx -y

sudo apt install git -y

sudo docker run hello-world

# cerbot
sudo apt install certbot -y

sudo apt install openssh-server -y

sudo apt install -y  python3-certbot-nginx


# yarn , nvm
curl -sSL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor -o /usr/share/keyrings/yarn-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/yarn-archive-keyring.gpg] https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list > /dev/null

sudo apt-get update -y

sudo apt-get install yarn -y


curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

source ~/.bashrc

# nodejs
nvm install 18.8.0

sudo apt install build-essential

sudo apt install sqlite3 -y



# ar.io reposunu klonluyoruz
git clone https://github.com/ar-io/ar-io-node

cd ar-io-node

Ar cüzdan kurup 2 tane cüzdan adres aliyoruz
> [ArConnect](https://www.arconnect.io/)
# 2 tane cüzdan oluşturun bunu aynı wallet içinde kurabilirsiniz ar connect kullanıyorsanız generate butonun basıp ikinci cüzdanınızı kurabilirsiniz birine mainnet token atacağız diğerinide observer olarak kullanacağız. 
Bu cüzdanlara marketten arweave token alip yolluyoruz. 0.5-1 token arasi yeterli olur.


#  env dosyasini editleyip asagidaki gibi yapiyoruz. 
nano .env

GRAPHQL_HOST=arweave.net
GRAPHQL_PORT=443
START_HEIGHT=1384000
RUN_OBSERVER=true
ARNS_ROOT_HOST=<domain-adresiniz.xyz>
AR_IO_WALLET=<Mainnet token atığınız cüzdan adresiniz>
OBSERVER_WALLET=<Observer olarak açtığınız ikinci cüzdan adresiniz>


# nodeumuzu calıstıralım:
sudo docker-compose up -d --build

sudo docker-compose logs -f --tail=0

# ipimizi teyit edelim ve ping atalım
curl ipinfo.io/ip

ip addr show | grep -w inet | awk '{print $2}' | awk -F'/' '{print $1}'

#Kendimize bir domain alip ar.io sunucusuna yönlendirme yapiyoruz.

![image](https://github.com/mesahin001/ar-io/assets/62426828/77a32636-8119-4cb5-ba40-37e237f28220)


#Letsencrypt ile sertifika aliyoruz
sudo certbot certonly --manual --preferred-challenges dns --email ******@hotmail.com -d aaaaa.xyz -d '*.aaaaaa.xyz'

Bu komutu calistirinca sana verdigi kodu txt olarak dns ayarlari sayfada ekleme yapiyoruz:

![image](https://github.com/mesahin001/ar-io/assets/62426828/415809eb-50bf-4a92-b139-9e573b9b7629)

5-10 dakika bekleyip daha sonra certbota onay veriyoruz.

#nginx config dosyasini edit yapiyoruz, icerde ne varsa silip asagidaki gibi yapiyoruz:



# Force redirects from HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name aaaa.xyz *.aaaa.xyz;
    return 301 https://aaaa.xyz$request_uri;

}

# Forward traffic to your node and provide SSL certificates
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name aaaa.xyz *.aaaa.xyz;

    ssl_certificate /etc/letsencrypt/live/aaaa.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/aaaa.xyz/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
    }
}


# Daha sonra nginx dosyayi kontrol edip servisi yeniden baslatalim:
sudo nginx -t

sudo service nginx restart

Kendi sayfamizi internet browserden acip kontrol ediyoruz

https://aaaa.xyz/

Sertifika hatasi vermeden böyle birsey gelmesi gerek:
{"network":"arweave.N.1","version":5,"release":68,"height":1385242,"current":"2tbqNmxc88z9Pzhlx5kKlwUOpk4hne23iMmLnFATDcQ2c-o7gtVKWJSg5twg6iMu","blocks":1385243,"peers":94,"queue_length":0,"node_state_latency":1}

Hersey tamamsa ar servisini yeniden baslatip log kontrolu yapiyoruz.

sudo docker-compose down

sudo docker-compose up -d --build

sudo docker-compose logs -f --tail=0


CTRL+C ile cikiyoruz.

# testnet-contract kurulumu:
git clone https://github.com/ar-io/testnet-contract

# dizine girip key.json'umuzu oluşturalım.
cd testnet-contract

nano key.json

Bu dosyanin icine ar connect ten ana cüzdanin key dosyasinin ciktisini alip icindekileri oldugu gibi yapistirip kaydediyoruz.

daha sonra ayni klasör icindeyken

yarn install

yaparak gerekli paketleri kuruyoruz.

tools/join-network.ts  dosyasini editleyip gerekli yerleri degistiriyoruz.

![image](https://github.com/mesahin001/ar-io/assets/62426828/65c77362-20a9-44d0-a341-c84ed7e8d925)

Daha sonra discord da #tesnet kanalina gidip ordan testnet icin basvuru yapiyoruz ve 23 soruyu cevaplayip tokenlerin gelmesini bekliyoruz.

tokenler geldikten sonra asagidaki komutu calistirip aga katiliyoruz.

yarn ts-node tools/join-network.ts

Bu komut size TX id: null verirse tokeniniz eksiktir, uzun bir TX verirse islem başarılı olmus demektir.


# Staking aktif etmek sunlari yapiyoruz.

cd /ar-io-node/testnet-contract/

sudo nano tools/update-gateway-settings.ts


Burda 3 noktada degisiklik yapiyoruz ve bastaki // isareti kaldiriyoruz

 const allowDelegatedStaking: boolean = true;
 
 const delegateRewardShareRatio: number = 10;
 
 const minDelegatedStake: number = 100;
 

Dosyanin alt kisminda bu ücü haric diger bütün degiskenleri önlerine // ekleyerek iptal ediyoruz.

![image](https://github.com/mesahin001/ar-io/assets/62426828/36b20ea9-5379-4955-99ba-fa1351e6ae10)



daha sonrada bu komutu calistiriyoruz:

yarn ts-node tools/update-gateway-settings.ts

# Baska birine yada kendinize delege etmek icin su dosyada degisiklik yapmak gerekiyor:

sudo nano tools/delegate-stake.ts

Burda sadece iki satirda degisilik yapilacak:

 const qty = 100;  //Ne kadarlik token stake etmek istiyorsaniz buraya yazin
 
 const target = 'l0iODykz8poPjVJ2***********************'; //Hangi cüzdan adresine stake yapacaksiniz buraya yaziyoruz


![image](https://github.com/mesahin001/ar-io/assets/62426828/fe6213ac-559b-4438-b051-3a792d1b7c7b)



daha sonrada bu komutu calistiriyoruz:

yarn ts-node tools/delegate-stake.ts



 
Commit 1 at README.md
Commit 2 at README.md
Commit 3 at README.md
Commit 4 at README.md
Commit 5 at README.md
Commit 6 at README.md
Commit 7 at README.md
Commit 8 at README.md
Commit 9 at README.md
Commit 10 at README.md
Commit 1 at README.md
Commit 2 at README.md
Commit 3 at README.md
Commit 4 at README.md
Commit 5 at README.md
Commit 6 at README.md
Commit 7 at README.md
Commit 8 at README.md
Commit 9 at README.md
Commit 10 at README.md
Commit 1 at README.md
Commit 2 at README.md
Commit 3 at README.md
Commit 4 at README.md
Commit 5 at README.md
Commit 6 at README.md
Commit 7 at README.md
Commit 8 at README.md
Commit 9 at README.md
Commit 10 at README.md
Commit 1 at README.md
