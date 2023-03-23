Sisestage seadistuste lattu ops.soft, käivitage `./ssl.sh` ja **ülemisse kataloogi** luuakse kaust `conf` .

## preambula

SMTP saab osta pilveteenuste pakkujatelt otse teenuseid, näiteks:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali pilve e-posti tõuge](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Samuti saate luua oma meiliserveri – piiramatu saatmine, madalad üldkulud.

Allpool demonstreerime samm-sammult oma meiliserveri loomist.

## Serveri valik

Isehostitav SMTP-server nõuab avalikku IP-aadressi, mille pordid 25, 456 ja 587 on avatud.

Tavaliselt kasutatavad avalikud pilved on need pordid vaikimisi blokeerinud ja võib-olla on võimalik neid avada töökäsu andmisega, kuid see on lõppude lõpuks väga tülikas.

Soovitan osta hostilt, millel on need pordid avatud ja mis toetab pöörddomeeninimede seadistamist.

Siin soovitan [Contabot](https://contabo.com) .

Contabo on Saksamaal Münchenis asuv hostingu pakkuja, mis asutati 2003. aastal väga konkurentsivõimeliste hindadega.

Valides ostuvaluutaks euro, on hind soodsam (8GB mälu ja 4 protsessoriga server maksab umbes 529 jüaani aastas ning esialgne paigaldustasu on aastaks tasuta).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Tellimuse esitamisel märkige, `prefer AMD` ja AMD protsessoriga server on parema jõudlusega.

Järgnevalt toon näitena Contabo VPS-i, et demonstreerida, kuidas oma meiliserverit ehitada.

## Ubuntu süsteemi konfiguratsioon

Siin on operatsioonisüsteem Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Kui ssh-server kuvab `Welcome to TinyCore 13!` (nagu alloleval joonisel näidatud), tähendab see, et süsteemi pole veel installitud. Katkestage ssh-ühendus ja oodake mõni minut, et uuesti sisse logida.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Kui kuvatakse `Welcome to Ubuntu 22.04.1 LTS` , on lähtestamine lõppenud ja saate jätkata järgmiste sammudega.

### [Valikuline] Initsialiseerige arenduskeskkond

See samm on valikuline.

Mugavuse huvides panin ubuntu tarkvara installimise ja süsteemi konfiguratsiooni saidile [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Käivitage ühe klõpsuga installimiseks järgmine käsk.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Hiina kasutajad, kasutage selle asemel järgmist käsku ja keel, ajavöönd jne määratakse automaatselt.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo lubab IPV6

Lubage IPV6, et SMTP saaks saata ka IPV6 aadressidega meile.

redigeeri `/etc/sysctl.conf`

Muutke või lisage järgmisi ridu

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Järgige [kontaboõpetust: IPv6-ühenduvuse lisamine serverisse](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Redigeerige `/etc/netplan/01-netcfg.yaml` , lisage paar rida, nagu on näidatud alloleval joonisel (Contabo VPS-i vaikekonfiguratsioonifailis on need read juba olemas, lihtsalt tühjendage need kommentaarid).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Seejärel `netplan apply` , et muudetud konfiguratsioon jõustuks.

Kui konfigureerimine on edukas, saate kasutada `curl 6.ipw.cn` , et vaadata oma välisvõrgu ipv6-aadressi.

## Kloonige konfiguratsioonihoidla operatsioonid

```
git clone https://github.com/wactax/ops.soft.git
```

## Looge oma domeeninime jaoks tasuta SSL-sertifikaat

Kirja saatmiseks on krüptimiseks ja allkirjastamiseks vaja SSL-sertifikaati.

Sertifikaatide genereerimiseks kasutame [faili acme.sh.](https://github.com/acmesh-official/acme.sh)

acme.sh on avatud lähtekoodiga automaatne sertifikaadi allkirjastamise tööriist,

Sisestage seadistuste lattu ops.soft, käivitage `./ssl.sh` ja **ülemisse kataloogi** luuakse kaust `conf` .

Leidke oma DNS-i pakkuja saidilt [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , muutke `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Seejärel käivitage `./ssl.sh 123.com` , et luua oma domeeninime jaoks sertifikaadid `123.com` ja `*.123.com` .

Esimesel käivitamisel installitakse [acme.sh](https://github.com/acmesh-official/acme.sh) automaatselt ja lisatakse automaatseks uuendamiseks ajastatud ülesanne. Näete `crontab -l` , seal on selline rida.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Loodud sertifikaadi tee on umbes `/mnt/www/.acme.sh/123.com_ecc。`

Sertifikaadi uuendamine kutsub esile skripti `conf/reload/123.com.sh` , muutke seda skripti, saate lisada käske, nagu `nginx -s reload` , et värskendada seotud rakenduste sertifikaadi vahemälu.

## Ehitage chasquidiga SMTP-server

[chasquid](https://github.com/albertito/chasquid) on avatud lähtekoodiga SMTP-server, mis on kirjutatud Go keeles.

Vanade meiliserveriprogrammide, nagu Postfix ja Sendmail, asendajana on chasquid lihtsam ja hõlpsamini kasutatav ning seda on lihtsam ka teisese arenduse jaoks.

Käivitage `./chasquid/init.sh 123.com` installitakse automaatselt ühe klõpsuga (asendage 123.com oma saatva domeeninimega).

## E-posti allkirja DKIM-i konfigureerimine

DKIM-i kasutatakse meiliallkirjade saatmiseks, et vältida kirjade käsitlemist rämpspostina.

Pärast käsu edukat käitamist palutakse teil määrata DKIM-kirje (nagu allpool näidatud).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Lihtsalt lisage oma DNS-i TXT-kirje (nagu allpool näidatud).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Vaadake teenuse olekut ja logisid

 `systemctl status chasquid` Vaadake teenuse olekut.

Normaalse töö olek on näidatud alloleval joonisel

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` või `journalctl -xeu chasquid` saab vaadata vealogi.

## Domeeninime vastupidine konfiguratsioon

Pöörddomeeninimi võimaldab IP-aadressi lahendada vastavaks domeeninimeks.

Pöörddomeeninime määramine võib takistada meilide tuvastamist rämpspostina.

Kui kiri on vastu võetud, teostab vastuvõttev server saatva serveri IP-aadressi domeeninime pöördanalüüsi, et kontrollida, kas saatval serveril on kehtiv pöörddomeeni nimi.

Kui saatval serveril ei ole pöörddomeeni nime või kui pöörddomeeni nimi ei ühti saatva serveri IP-aadressiga, võib vastuvõttev server meili rämpspostiks tunnistada või selle tagasi lükata.

Külastage aadressi [https://my.contabo.com/rdns](https://my.contabo.com/rdns) ja seadistage, nagu allpool näidatud

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Pärast pöörddomeeninime määramist ärge unustage konfigureerida domeeninime ipv4 ja ipv6 edasilahutusvõimet serverile.

## Redigeerige faili chasquid.conf hostinime

Muutke `conf/chasquid/chasquid.conf` pöörddomeeni nime väärtuseks.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Seejärel käivitage teenuse taaskäivitamiseks `systemctl restart chasquid` .

## Varunda conf git hoidlasse

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Näiteks varundan conf-kausta enda githubi protsessi järgmiselt

Esmalt looge eraladu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Sisestage conf kataloog ja esitage lattu

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Lisa saatja

jooksma

```
chasquid-util user-add i@wac.tax
```

Saab lisada saatja

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Veenduge, et parool on õigesti seadistatud

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Pärast kasutaja lisamist uuendatakse `chasquid/domains/wac.tax/users` , ärge unustage seda lattu esitada.

## DNS lisa SPF-kirje

SPF (Sender Policy Framework) on meilikontrolli tehnoloogia, mida kasutatakse meilipettuste vältimiseks.

See kontrollib kirja saatja identiteeti, kontrollides, kas saatja IP-aadress ühtib selle domeeninime DNS-kirjetega, milleks ta väidetavalt kuulub, vältides petturitel võltskirjade saatmist.

SPF-kirjete lisamine võib takistada meilide tuvastamist rämpspostina nii palju kui võimalik.

Kui teie domeeninimeserver ei toeta SPF-tüüpi, lisage lihtsalt TXT-tüüpi kirje.

Näiteks faili `wac.tax` SPF on järgmine

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF `_spf.wac.tax` jaoks

`v=spf1 a:smtp.wac.tax ~all`

Pange tähele, et olen siin `include:_spf.google.com` , sest ma konfigureerin hiljem Google'i postkasti saatmisaadressiks `i@wac.tax` .

## DNS-i konfiguratsioon DMARC

DMARC on lühend sõnast (domeenipõhine sõnumi autentimine, aruandlus ja vastavus).

Seda kasutatakse SPF-i tagasilöökide jäädvustamiseks (võib-olla põhjustatud konfiguratsiooniveadest või keegi teine ​​teeskleb, et olete rämpsposti saatmiseks).

Lisage TXT-kirje `_dmarc` ,

Sisu on järgmine

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Iga parameetri tähendus on järgmine

### p (poliitika)

Näitab, kuidas käsitleda e-kirju, mille SPF (Sender Policy Framework) või DKIM (DomainKeys Identified Mail) kinnitamine ebaõnnestub. Parameetri p saab määrata ühele kolmest väärtusest:

* none: midagi ei võeta, ainult kinnitustulemus edastatakse saatjale e-posti aruandlusmehhanismi kaudu.
* Karantiin: asetage kontrollimata kirjad rämpsposti kausta, kuid ei lükka kirja otse tagasi.
* tagasilükkamine: keelduge otse meilidest, mille kinnitamine ebaõnnestub.

### fo (tõrgete valikud)

Määrab aruandlusmehhanismi tagastatava teabe hulga. Selle saab määrata ühele järgmistest väärtustest:

* 0: teatage kõigi sõnumite valideerimise tulemused
* 1: teatage ainult sõnumitest, mille kinnitamine ebaõnnestub
* d: teatage ainult domeeninime kinnitamise tõrgetest
* s: teatage ainult SPF-i kinnitamise tõrgetest
* l: teatage ainult DKIM-i kinnitamise tõrgetest

### rua & ruf

* rua (Koondaruannete aruandluse URI): koondaruannete saamise e-posti aadress
* ruf (kohtuekspertiisi aruannete aruandluse URI): e-posti aadress üksikasjalike aruannete saamiseks

## Lisage MX-kirjed, et edastada e-kirjad Google Maili

Kuna ma ei leidnud tasuta ettevõtte postkasti, mis toetaks universaalseid aadresse (Catch-All, saab vastu võtta kõiki sellele domeeninimele saadetud e-kirju, ilma eesliidete piiranguteta), kasutasin chasquid'i kõigi kirjade edastamiseks oma Gmaili postkasti.

**Kui teil on oma tasuline ettevõtte postkast, ärge muutke MX-i ja jätke see samm vahele.**

Redigeeri `conf/chasquid/domains/wac.tax/aliases` , määra edastamise postkast

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` tähistab kõiki e-kirju, `i` on ülaltoodud saatva kasutaja e-posti aadressi eesliide. Kirjade edastamiseks peab iga kasutaja lisama rea.

Seejärel lisage MX-kirje (osutan siin otse vastupidise domeeninime aadressile, nagu on näidatud alloleva joonise esimesel real).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Kui konfiguratsioon on lõpule viidud, saate kasutada teisi e-posti aadresse, et saata e-kirju aadressile `i@wac.tax` ja `any123@wac.tax` , et näha, kas saate Gmailis meile vastu võtta.

Kui ei, kontrollige chasquid logi ( `grep chasquid /var/log/syslog` ).

## Saatke e-kiri Google Mailiga aadressile i@wac.tax

Pärast seda, kui Google Mail kirja sai, lootsin loomulikult vastata i.wac.tax@gmail.com asemel `i@wac.tax` .

Külastage aadressi [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) ja klõpsake käsul „Lisa teine ​​e-posti aadress“.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Seejärel sisestage kinnituskood, mis saadeti e-kirjaga, millele saadeti edasi.

Lõpuks saab selle määrata saatja vaikeaadressiks (koos võimalusega vastata sama aadressiga).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Sel viisil oleme lõpetanud SMTP meiliserveri loomise ja samal ajal kasutame Google Maili meilide saatmiseks ja vastuvõtmiseks.

## Saatke testmeil, et kontrollida, kas konfiguratsioon on edukas

Sisestage `ops/chasquid`

Käivitage `direnv allow` installida sõltuvusi (direnv on installitud eelmises ühe võtmega lähtestamisprotsessis ja kestale on lisatud konks)

siis jookse

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Parameetrite tähendus on järgmine

* kasutaja: SMTP kasutajanimi
* pass: SMTP parool
* kellele: saaja

Võite saata testmeili.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Testmeilide saamiseks on soovitatav kasutada Gmaili, et kontrollida, kas konfiguratsioonid on edukad.

### TLS-i standardne krüptimine

Nagu on näidatud alloleval joonisel, on see väike lukk, mis tähendab, et SSL-sertifikaat on edukalt lubatud.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Seejärel klõpsake "Kuva algne e-kiri"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Nagu on näidatud alloleval joonisel, kuvatakse Gmaili algse meili lehel DKIM, mis tähendab, et DKIM-i konfigureerimine on edukas.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Kontrollige algse meili päises käsku Saadud ja näete, et saatja aadress on IPV6, mis tähendab, et ka IPV6 on edukalt konfigureeritud.
