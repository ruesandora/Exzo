<h1 align="center"> Exzo Network </h1>

> Exzo'yu hasta olduğum dönem formunu doldurmuştum, önemi olmayacağını biliyordum ki öylede oldu ve hepiniz katılabilirsiniz.

> Ödül şu şekilde: Node'un testnet bitene kadar `%95 ve üzerinde uptime` alırsa ödül alırsın almazsa geçmiş olsun.

> Ne kadar sürecek bilgi yok ama kısa sürecek formda yazıyordu.

> `Uptime'dan` ötürü iyi bir sunucu tercih edin.

> Ricamdır adım adım gidin, `acelemiz yok` kaçan bir şey yok.

> Bu bilgiler değişkenlik gösterebilir benim okuduklarım bu şekilde sizde araştırınız.

>  Topluluk kanalları: [Duyurular](https://t.me/RuesAnnouncement) - [Sorularınız İçin](https://t.me/RuesChat) -  [Explorer](https://exzoscan.io/?cluster=testnet)

<h1 align="center"> Donanım: </h1>

> Sunucu olarak [Hetzner Kullanıyorum](https://github.com/ruesandora/Hetzner/blob/main/README.md)

```sh
# Benim kullandığım: 
4 CPU - 8 RAM - 150 SSD - 20.04 Ubuntu
# Ki bence bir tık altı daha sunucuya kurulabilir.

# Dökümasyon önerisi:
8 CPU - 32 RAM - 500 SSD
```

<h1 align="center"> Güncellemeler ve Gerekli paketler </h1>

```sh
# sistem güncellemeleri ve gerekli kütüphaneler.
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install libudev-dev
sudo apt-get install protobuf-compiler
sudo apt-get install pkg-config
sudo apt-get install libclang-dev
sudo apt-get install build-essential

# nodejs'i yüklüyoruz
sudo apt-get install nodejs

# Rust'ı yüklüyoruz:
curl https://sh.rustup.rs -sSf | sh
# Rust'ı kurarken seçenekler arasından 1'i seçiyoruz.

source $HOME/.cargo/env

rustup component add rustfmt
rustup update
```

<h1 align="center"> Exzo Kurulumu </h1>

> Node-Validator isminde bir dizin oluşturuyoruz

```sh
cd
mkdir Node-Validator
cd Node-Validator
```

> `Exzo`'yu klonluyoruz

```sh
git clone https://github.com/ExzoNetwork/Exzo-Network-Blockchain.git
cd Exzo-Network-Blockchain
```

> Hata almamak için paketleri güncelliyoruz

```sh
sudo apt-get install libudev-dev
sudo apt-get install protobuf-compiler
sudo apt-get install pkg-config
sudo apt-get install libclang-dev
sudo apt install build-essential
```

> Relase işlemi `10-15 dakika` sürebilir.

```sh
# release sonunda RUST_BACKTRACE=1 hatası alırsanız üstte ki 5 kodu tekrar girin ve tekrar release edin.
cargo build --release 
cd target/release
```

> Exzo için `config.yml`'ı oluşturuyoruz
```sh
./exzo config set --url https://rpc-test-1.exzo.network/rpc
```

> Bu komutun çıktısı `websocket` , `RPC url` gibi bazı bilgiler gösterecek, yani doğru çıktı.
```sh
./exzo config get
```

> Komutları tek tek girdiğimizde ufak çıktılar alacağız. `version ve transaction`
```sh
./exzo transaction-count
./exzo cluster-version
```

> Son olarak bu komutta loglarda `IP'ımızı` görüyorsak `ctrl + c` ile durdurabiliriz.
```sh
./exzo-gossip spy --entrypoint bootnode-test.exzo.network:8001
```

<h1 align="center"> Cüzdan işlemleri </h1>

> DİKKAT: Bir metin belgesi açın ve 3 farklı private key / cüzdan adresimiz olacak hepsini kaydedin.

```sh
> validator-keypair ==> Cüzdan-1
> vote-keypair ==> Cüzdan-2
> withdrawer-keypair ==> Cüzdan-3
```

> `Cüzdan-1`'i oluşturuyoruz:

```sh
./exzo-keygen new -o ~/new-validator-keypair.json

# Çıkan çıktıyı kaydediyoruz ve altta ki komut ile cüzdan adresimiz aynı mı kontrol ediyoruz:
./exzo-keygen pubkey ~/new-validator-keypair.json

# Son olarak bu komutla yine RPC url ve websocket gibi çıktıları alıyoruz:
./exzo config set --keypair ~/new-validator-keypair.json

# Cüzdan-1'e test tokeni alıyoruz, < > tırnaklar arasına cüzdan-1 adresi giriyoruz. (< > silinecek)
./exzo airdrop 1 <Cüzdan-1> --url https://rpc-test-1.exzo.network/rpc

# 1 Token gelmiş mi bakıyoruz:
./exzo balance <Cüzdan-1>
```

> `Cüzdan-2`'i oluşturuyoruz:

```sh
./exzo-keygen new -o ~/my-vote-account-keypair.json

# Yine adresimizi kontrol ediyoruz:
./exzo-keygen pubkey ~/my-vote-account-keypair.json
```

> `Cüzdan-3`'i oluşturuyoruz:
```sh
./exzo-keygen new -o ~/withdrawer-keypair.json

# Yine adresimizi kontrol ediyoruz:
./exzo-keygen pubkey ~/withdrawer-keypair.json
```

> Son olarak `vote account` oluşturup, `vote key`'e token transfer ediyoruz.

```sh
# Cüzdan-3 eklenecek burada.
./exzo create-vote-account ~/my-vote-account-keypair.json ~/new-validator-keypair.json <Cüzdan-3> --commission 1 

# Cüzdan-2 eklenecek burada.
./exzo transfer --from ~/new-validator-keypair.json <cuzdan2> 0.5 --url https://rpc-test-1.exzo.network/rpc --fee-payer ~/new-validator-keypair.json
```

<h1 align="center"> Validatörü başlatma </h1>

```sh
screen -S exzo

# Soracağınızı biliyorum, değişmeniz gereken bir değer yok olsa söylerdim :)
./exzo-validator --identity ~/new-validator-keypair.json --vote-account ~/my-vote-account-keypair.json --ledger /root/exzonode/ledger/ --rpc-port 8899 --dynamic-port-range 8000-8012 --entrypoint bootnode-test.exzo.network:8001 --limit-ledger-size --expected-shred-version 17211 --max-genesis-archive-unpacked-size 707374182 --log -
```

> validatörü başlatırken port sorun yaşarsanız [buradan](https://github.com/ruesandora/Exzo/blob/main/port-sorunu.md) çözebilirsiniz.

```
# CTRL A D ile çıkıyoruz ve logları kontrol ediyoruz
./exzo-gossip spy --entrypoint bootnode-test.exzo.network:8001
```

> Doğru çalıştığını nasıl anlarımın cevabı görselde:

![image](https://github.com/ruesandora/Exzo/assets/101149671/bfd6cba6-0e5b-4b16-b677-e0d8c6decbd5)

> Repoyu sağ üstten fork/yıldızlayabilirsiniz.

> Repo hakkında daha detaylı bilgileri yarın eklerim.
