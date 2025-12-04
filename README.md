# Ente setup guide for Raspberry Pi

## set up pi
This guide assumes a Raspberry Pi OS Lite image.

Flash this image using the Raspberry pi imager.

## set up docker
```sh
# 1. Prep
sudo rm -f /etc/apt/sources.list.d/docker.list /etc/apt/keyrings/docker*

# 2. Install prereqs
sudo apt-get update
sudo apt-get install -y ca-certificates curl

# 3. Add Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 4. Add Docker repo 
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Install Docker + Compose v2 plugin
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 6. Start Docker
sudo systemctl enable --now docker

# 7. Add your user to docker group
sudo usermod -aG docker $USER
newgrp docker
```


## Install ente

Visit https://ente.io/help/self-hosting/ for more details!

`sh -c "$(curl -fsSL https://raw.githubusercontent.com/ente-io/ente/main/server/quickstart.sh)"`

### Fix uploads
After this, uploads won't work if you access the raspberry pi from another machine. To enable this, make sure the `museum.yaml` in `my-ente`
contains your IP address instead of `localhost`. Also disable fast uploads in the web UI.

### Fix storage limits
Next, you will run into a storage limit. I was unable to use the CLI, if that is the case for you, change this in SQL:

`docker exec -it my-ente-postgres-1 psql -U pguser -d ente_db` (pay attention to use the right db name, try something like `docker stats` to see them)

Get your user id:

```sql
SELECT
    user_id,
    name,
    to_timestamp(creation_time / 1000000) AS account_created   -- converts microseconds → readable date
FROM users
ORDER BY creation_time;
```

then run `INSERT INTO storage_bonus (bonus_id, user_id, storage, type, valid_till) VALUES ('self-hosted-myself', (SELECT user_id FROM users LIMIT 1), 1099511627776, 'ADD_ON_SUPPORT', 0);`
