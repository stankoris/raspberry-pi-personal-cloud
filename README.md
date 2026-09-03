Install pi installer (ovde daj korake za linux i za windows)
daj korake, naglai da izaberu rasppbery pi OS (Legacy, 64-bit) lite umesto rasppbery pi OS lite (64-bit) jer ce im prvi praviti probleme

<img width="860" height="605" alt="image" src="https://github.com/user-attachments/assets/2668bdba-9b00-4761-96e4-fda76c83f5d1" />

ssh na ssh cloudadmin@pi-cloud.local

Step 2 — Update Raspberry Pi
sudo apt update

Osvežava listu dostupnih paketa.

sudo apt upgrade -y

Step 3 — Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

Instalira Tailscale na Raspberry Pi.

Zatim:

sudo tailscale up

Pokreće Tailscale i prikazuje URL za autentifikaciju.

Otvori prikazani URL na laptopu i prijavi Pi na svoj Tailscale nalog.

Step 4 — Verify Tailscale
ping <TAILSCALE_IP_PI-ja>

Zatim:

ssh strnga@<TAILSCALE_IP_PI-ja>

Na primer:

ssh strnga@100.80.20.15

Ako uđeš, ovaj deo projekta je završen:

Laptop
   │
   │ Tailscale
   ▼
pi-cloud
   │
   └── SSH

Kad potvrdiš da ovo radi, idemo direktno na SFTPGo i /srv/cloud storage direktorijum.

