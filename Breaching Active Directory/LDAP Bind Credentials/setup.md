

## ✅ 1. OpenLDAP paketlarini o‘rnatish

```bash
sudo pacman -Syu openldap
```

---

## ✅ 2. Ma'lumotlar katalogini tayyorlash

```bash
sudo mkdir -p /var/lib/openldap/openldap-data
sudo chown -R ldap:ldap /var/lib/openldap
```

---

## ✅ 3. `slapd.conf` faylini yaratish

```bash
sudo nano /etc/openldap/slapd.conf
```

Ichiga quyidagilarni yozing (moslashtirilgan):

```conf
include         /etc/openldap/schema/core.schema

pidfile         /run/openldap/slapd.pid
argsfile        /run/openldap/slapd.args

modulepath      /usr/lib/openldap
moduleload      back_mdb

database        mdb
maxsize         1073741824
suffix          "dc=hacknow,dc=uz"
rootdn          "cn=admin,dc=hacknow,dc=uz"
rootpw          {SSHA}R+0lLB66ybexnaln3wkBglc+yTVTdEn9
directory       /var/lib/openldap/openldap-data
```

> Parol: `password11` ga mos hash `{SSHA}...` bo‘lib turibdi. `slappasswd` bilan xohlagancha o‘zgartirishingiz mumkin.

---

## ✅ 4. Dynamic config (`slapd.d`) yaratish

```bash
sudo mkdir -p /etc/openldap/slapd.d
sudo chown -R ldap:ldap /etc/openldap/slapd.d

sudo slaptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d
```

---

## ✅ 5. Ruxsatlarni to‘g‘rilash

```bash
sudo chown -R ldap:ldap /etc/openldap/slapd.d
sudo chmod -R 750 /etc/openldap
```

---

## ✅ 6. `slapd` serverni ishga tushirish (manual)

```bash
sudo slapd -F /etc/openldap/slapd.d -h "ldap:/// ldapi:///" -u ldap -g ldap -d 1
```

Agar hamma narsa to‘g‘ri bo‘lsa, `slapd startup: initiated.` chiqadi.

---

## ✅ 7. Admin sifatida ulanishni test qilish

```bash
ldapsearch -H ldap://localhost -x -D "cn=admin,dc=hacknow,dc=uz" -W -b "dc=hacknow,dc=uz"
```

---

## ✅ 8. Agar doimiy xizmat kerak bo‘lsa, systemd unit yaratish:

```ini
# /etc/systemd/system/slapd.service
[Unit]
Description=OpenLDAP server daemon
After=network.target

[Service]
Type=forking
ExecStart=/usr/bin/slapd -F /etc/openldap/slapd.d -h "ldap:/// ldapi:///"
User=ldap
Group=ldap

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now slapd
```

---

## 🔐 9. SASL mexanizmini faqat `PLAIN`, `LOGIN` bilan cheklash (ixtiyoriy)

`olcSaslSecProps.ldif` bilan:

```ldif
dn: cn=config
changetype: modify
replace: olcSaslSecProps
olcSaslSecProps: noanonymous,minssf=0,passcred
```

```bash
ldapmodify -H ldap://localhost -x -D "cn=admin,cn=config" -W -f olcSaslSecProps.ldif
```

---

## 🧠 Yakuniy tavsiyalar:

| Qadam              | Tavsiya                                                                               |
| ------------------ | ------------------------------------------------------------------------------------- |
| 🧱 Bazani yaratish | `ldapadd` orqali yangi foydalanuvchi, OU va gruppalarni yarating                      |
| 🔁 Zaxira          | `slapcat`, `slapadd`, `slapmodify` orqali konfiguratsiya va ma’lumotlar bilan ishlang |
| 🔐 Xavfsizlik      | TLS/SSL, access rules (`olcAccess`) qo‘llang                                          |
| 📂 GUI             | `phpldapadmin`, `ldap-account-manager` bilan ishlash osonlashadi                      |

