# QChain Setup Guide (Ubuntu / Fedora · x86_64 / ARM64)

This guide provides step-by-step instructions for setting up the entire QChain backend and blockchain infrastructure on **Ubuntu** (20.04/22.04/24.04) or **Fedora** Linux, on both **x86_64 (AMD/Intel)** and **ARM64 (Apple Silicon M-series via UTM / AWS Graviton)** architectures.

> **ARM64:** the `qchain-pqc/*` Fabric images in section 6 are built for x86_64, so an ARM64 machine needs its own build of them.
>
> **Repositories.** Since 9 November 2026 the code lives in separate repositories under `teamqchain`:
> [qchain-network](https://github.com/teamqchain/qchain-network) (the Fabric network),
> [qchain-backend](https://github.com/teamqchain/qchain-backend) (the Go API, database schema, gateway and CI/CD),
> [qchain-portal](https://github.com/teamqchain/qchain-portal), [qchain-wallet](https://github.com/teamqchain/qchain-wallet)
> and [qchain-shared](https://github.com/teamqchain/qchain-shared) (the Flutter apps). This guide sets up the server side
> from the first two, cloned side by side.

---

## 1. Prerequisites

### Step 1A: For Ubuntu / Debian

```bash
# Update package lists
sudo apt update && sudo apt upgrade -y

# 1. Install system utilities
sudo apt install -y curl wget git tar build-essential ca-certificates gnupg

# 2. Install Docker & Docker Compose
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# (Note: log out and log back in or run 'newgrp docker' for group permissions to take effect)

# 3. Install Node.js 20.x & npm (only for the chaincode unit tests)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# 4. Install MySQL Server (the VM runs MySQL 8.x, not MariaDB — its JSON-column
#    behavior differs from MariaDB's in ways the schema relies on)
sudo apt install -y mysql-server
sudo systemctl enable --now mysql
```

### Step 1B: For Fedora / RHEL

```bash
# Update system
sudo dnf update -y

# 1. Install system utilities
sudo dnf install -y curl wget git tar dnf-plugins-core

# 2. Install Docker & Docker Compose
sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo usermod -aG docker $USER

# 3. Install Node.js & npm (only for the chaincode unit tests)
sudo dnf install -y nodejs npm

# 4. Install MySQL Server (the VM runs MySQL 8.x, not MariaDB)
sudo dnf install -y mysql-server
sudo systemctl enable --now mysqld
```

---

### Step 1C: Install IPFS (Kubo) (Universal)

Run these commands on either OS. Architecture (`x86_64` vs `arm64`) is detected automatically:

```bash
# Detect architecture
ARCH=$(uname -m)
if [ "$ARCH" = "x86_64" ]; then
    KUBO_ARCH="amd64"
elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then
    KUBO_ARCH="arm64"
else
    echo "Unsupported architecture: $ARCH"; exit 1
fi

# 1. Install IPFS (Kubo)
wget "https://dist.ipfs.tech/kubo/v0.32.1/kubo_v0.32.1_linux-${KUBO_ARCH}.tar.gz"
tar -xvzf "kubo_v0.32.1_linux-${KUBO_ARCH}.tar.gz"
cd kubo && sudo bash install.sh && cd ..
rm -rf kubo "kubo_v0.32.1_linux-${KUBO_ARCH}.tar.gz"

# 2. Verify installations
docker ps
mysql --version
ipfs --version
node -v
```

The Fabric tools run from Docker images, so the host needs no Fabric binaries (see section 6).

## 2. Clone the Repositories

The backend reads the network's wallets and certificates from the `qchain-network` clone **next to** it, so clone both
into one folder (on the project VM: `~/Desktop/QChain/teamqchain`):

```bash
mkdir -p ~/teamqchain && cd ~/teamqchain
git clone https://github.com/teamqchain/qchain-network.git
git clone https://github.com/teamqchain/qchain-backend.git
export NET=$PWD/qchain-network API=$PWD/qchain-backend
```

## 3. Configure Local Hosts Mappings (/etc/hosts)

The Fabric CLIs and the backend reach the orderer and peers by name, so map these hostnames to `127.0.0.1` in `/etc/hosts`. The CAs are reached at `https://localhost:<port>`, so they need no entry:

```bash
sudo tee -a /etc/hosts <<EOF

# QChain Fabric Domains
127.0.0.1 peer0.government.uae.com
127.0.0.1 peer0.general.uae.com
127.0.0.1 orderer0.orderer.example.com
EOF
```

## 4. Database Initialization

We need to create the database, a specific user for the Go backend, and seed the initial schema.

```bash
# Secure MySQL (optional but recommended)
sudo mysql_secure_installation

# Create Database and User
sudo mysql -e "CREATE DATABASE qchain_db CHARACTER SET utf8mb4;"
sudo mysql -e "CREATE USER 'qchain_user'@'%' IDENTIFIED BY 'password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON qchain_db.* TO 'qchain_user'@'%';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Seed the schema (includes every column the backend needs — no separate migrations to run)
sudo mysql qchain_db < "$API/db/schema.sql"
```

## 5. IPFS Setup

```bash
ipfs init
bash "$API/scripts/setup-ipfs-service.sh"
```

## 6. Hyperledger Fabric Network Setup

QChain's network uses a post-quantum Membership Service Provider (MSP): the orderer, peers, admins and application users all hold ML-DSA-44 identities, and the peer and orderer TLS certificates are ML-DSA-44 too. The [qchain-network README](https://github.com/teamqchain/qchain-network#readme) explains how it works, the ports, the limits and the ledger reset.

### Load the Images and Install the CLI Wrappers

Stock `hyperledger/*` images can't run ML-DSA identities, so the network runs from five images built from the team's Fabric and Fabric CA forks:

| Image | Role |
|---|---|
| `qchain-pqc/orderer:e472b4ea` | Orderer |
| `qchain-pqc/peer:e472b4ea` | Peers, including the chaincode-as-a-service builder |
| `qchain-pqc/fabric-ca-server:e82caf43` | Fabric CA, one per organisation |
| `qchain-pqc/ccaas-qchain:2.0` | QChaincode, run as a service |
| `qchain-pqc/tools:20261105` | CLIs: `peer`, `osnadmin`, `fabric-ca-client`, `cryptogen`, `configtxgen`, `mkwallet-g1` |

Load them first and check with `docker images 'qchain-pqc/*'`. The CLIs run from the tools image through wrappers:

```bash
bash "$NET/scripts/qchain-pqc-cli.sh" install ~/pqc-tools/bin
```

### Build the Network

```bash
cd "$NET"
bash scripts/generate-crypto-pqc.sh     # ML-DSA-44 crypto material, CA files, ECDSA CA server TLS
bash scripts/start-fabric-network.sh    # CAs, orderer, peers
sleep 10
bash scripts/create-channel-pqc.sh      # channel mychannel: genesis, orderer join, peer joins
bash scripts/registerEnroll.sh          # CA admins, issuer1 and verifier1 at GeneralCA, wallets
bash scripts/deployCC-pqc.sh            # chaincode qchaincode 2.0 as a service
```

The containers use Docker's `unless-stopped` restart policy, so they come back by themselves after a Docker or machine restart.

To empty the ledger later, follow the reset in the [qchain-network README](https://github.com/teamqchain/qchain-network#clean-ledger-reset). The ledger and MySQL must always be cleared together.

### Check That the Identities Are ML-DSA-44

```bash
# Older OpenSSL prints the OID 2.16.840.1.101.3.4.3.17; newer versions print ML-DSA-44. Nothing else should appear.
find "$NET/crypto-material" -path '*/signcerts/*.pem' -exec openssl x509 -in {} -noout -text \; | grep 'Public Key Algorithm' | sort | uniq -c
```

### Check the Chaincode

```bash
# Smoke test: an empty batch read must answer {"holders":{}}
( . "$NET/scripts/env-gov.sh"; "$PEER" chaincode query -C mychannel -n qchaincode -c '{"Args":["getHolders","[]","summary"]}' )

# Unit tests against an in-memory ledger; all tests must pass
cd "$NET/chaincode"
npm install
node --test
```

The chaincode runs from the `qchain-pqc/ccaas-qchain` image. A chaincode change means rebuilding that image and upgrading the chaincode definition with the next sequence on both peers. This has not been done on this network yet.

## 7. Backend (Go API) Setup

Because `liboqs-go` requires compiling the C library `liboqs` with CGo, we build the Docker image first (which compiles `liboqs` inside Docker automatically), and then run `keygen` from the Docker image.

```bash
cd "$API"

# 1. Run the Go unit tests in the image's builder stage, then build the image
#    (first build takes ~15-20 mins to compile liboqs; go vet also runs during the build)
docker build --target builder -f docker/Dockerfile -t qchain-api:test .
docker run --rm qchain-api:test go test -count=1 ./...
bash docker/docker-build.sh

# 2. Generate organisation ML-DSA-44 keys via Docker
docker run --rm qchain-api:latest keygen

# 3. Create the .env configuration file (repository root; .env.example is the full template)
cat > .env <<'ENV'
ISSUER_PRIVATE_KEY_HEX=<insert-private-key-hex-here>
ISSUER_PUBLIC_KEY_HEX=<insert-public-key-hex-here>
ISSUER_ORG_ID=GeneralMSP
ISSUER_ORG=general
ISSUER_IDENTITY=issuer1
VERIFIER_ORG=general
VERIFIER_IDENTITY=verifier1
MYSQL_DSN=qchain_user:password@tcp(127.0.0.1:3306)/qchain_db
NETWORK_ROOT=/qchain-network
GOV_PEER_ENDPOINT=peer0.government.uae.com:27051
GEN_PEER_ENDPOINT=peer0.general.uae.com:29051
CHANNEL_NAME=mychannel
CHAINCODE_NAME=qchaincode
IPFS_HOST=127.0.0.1:5001
SERVER_PORT=3000
ENV
chmod 600 .env     # it holds the private key

# 4. Open .env in your text editor and insert the hex keys you generated in step 2

# 5. Start the backend container (it mounts ../qchain-network read-only)
bash docker/docker-run.sh

# 6. Seed the demo holders on Fabric via the API
bash scripts/setup-demo.sh http://localhost:3000

# 7. (Track H) Register holder public keys (wallet activation)
# The QWallet mobile app generates keys locally on-device and binds the public keys on-chain via
# /mobile/registerHolderKeys. A credential can only be issued to a holder whose keys are bound.
# You can check if a holder has keys registered (read from the chain):
curl -s "http://localhost:3000/mobile/checkKeys?emiratesID=784-1990-1234567-1"

# 8. End-to-end check (plays issuer + holder with the keygen tool; needs IPFS, Fabric and MySQL up)
bash tests/e2e_api_test.sh http://localhost:3000
```

> **Where data lives.** The ledger is the source of truth for every field it holds (holder names and
> public keys, credential type/status/dates, CID, field hashes, issuer signature); IPFS holds the only
> copy of each credential's body, encrypted to the holder; MySQL holds only what exists nowhere else
> (Emirates ID → holder mapping, email, display IDs, sessions, subscriptions, logs). IPFS must be running
> for issuance, and Fabric for every read — there is no database fallback.

## 8. Cloudflare Tunnel Setup (Free Random URL)

> **This is for a temporary, single-session tunnel straight to your local backend** (e.g. quickly
> sharing `localhost:3000` with a frontend developer while you're both online). It is not how the
> live system is actually exposed — the production deployment (see the
> [qchain-backend gateway README](https://github.com/teamqchain/qchain-backend/tree/main/gateway)) uses a **Tailscale Funnel**
> instead, specifically because a Cloudflare quick tunnel's URL is random and doesn't survive a
> restart, whereas Tailscale Funnel gives a permanent `*.ts.net` address that comes back automatically
> after a reboot. If you want a stable, shareable link (not just a one-off dev session), follow
> `bash "$API/gateway/setup-tailscale-funnel.sh"` instead, as described in the qchain-backend README's
> "Public access" step.

To instantly share the backend API with your frontend developer without creating a Cloudflare account or buying a domain, you can use Cloudflare's free Quick Tunnel:

```bash
# 1. Download the standalone cloudflared binary (auto-detects architecture)
ARCH=$(uname -m)
CF_ARCH="amd64"
if [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then
    CF_ARCH="arm64"
fi
curl -L --output cloudflared "https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-${CF_ARCH}"
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/

# 2. Start the quick tunnel pointing to port 3000
cloudflared tunnel --url http://localhost:3000
```

This will print an instant, free public URL in your terminal:
```text
+--------------------------------------------------------------------------------------------+
|  Your quick Tunnel has been created! Visit it at (it may take some time to be reachable):  |
|  https://random-words-here.trycloudflare.com                                               |
+--------------------------------------------------------------------------------------------+
```

Give `https://random-words-here.trycloudflare.com` to your frontend developer to use as the `API_BASE_URL`.

## 9. Frontend Setup

The two apps have their own repositories, and both use the shared package in
[qchain-shared](https://github.com/teamqchain/qchain-shared):
- **QPortal ([qchain-portal](https://github.com/teamqchain/qchain-portal))**: a Flutter **web** app for Issuers, Verifiers, and IT Administrators.
- **QWallet ([qchain-wallet](https://github.com/teamqchain/qchain-wallet))**: a Flutter **mobile-only** app for credential holders to view, store, and
  present credentials. It cannot run in a browser — its ML-KEM-768/ML-DSA-44 key pair is generated and
  held only on-device (a browser build would mean a different key pair per browser/device, breaking the
  one-key-pair-per-holder model), and its PQC library (`liboqs`) uses `dart:ffi`, which doesn't exist in
  a browser at all. Run it on a device/emulator with `flutter run`, or build an install
  (`flutter build apk` / `flutter build ios`).

QPortal's public deployment is on Vercel (https://qportal.vercel.app; see the qchain-portal README). For development, run the apps with Flutter:

---

### Method A: Running Locally with Flutter SDK (Development)

Ensure you have Flutter and Google Chrome installed on your machine (`flutter doctor`).

#### 1. Connected to Local Backend (`http://localhost:3000`)

If the backend is running locally on port 3000, simply run:

```bash
# Run QPortal (Issuer/Verifier/Admin Portal)
git clone https://github.com/teamqchain/qchain-portal.git && cd qchain-portal
flutter pub get
flutter run -d chrome

# In a separate terminal, run QWallet (Holder App) on a connected device/emulator —
# QWallet cannot target chrome/web, see the note above.
git clone https://github.com/teamqchain/qchain-wallet.git && cd qchain-wallet
flutter pub get
flutter devices        # pick a device id
flutter run -d <device-id>
```

#### 2. Connected to Remote Backend (via Cloudflare Tunnel URL)

If connecting to a remote backend exposed by Cloudflare Tunnel, inject the URL at startup:

```bash
# Run QPortal (in the qchain-portal clone)
flutter pub get
flutter run -d chrome --dart-define=API_BASE_URL=https://<your-tunnel-name>.trycloudflare.com

# Run QWallet (device/emulator, in the qchain-wallet clone)
flutter pub get
flutter run -d <device-id> --dart-define=API_BASE_URL=https://<your-tunnel-name>.trycloudflare.com
```

#### 3. VS Code One-Click Launch Configuration (`launch.json`)

To run and debug directly in VS Code, open the folder holding both clones and create or edit `.vscode/launch.json`. QWallet's `deviceId` is
machine-specific (run `flutter devices` and use the id it prints) since it can't target `chrome`/web:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "QPortal (Local)",
      "cwd": "qchain-portal",
      "request": "launch",
      "type": "dart",
      "deviceId": "chrome"
    },
    {
      "name": "QPortal (Cloudflare)",
      "cwd": "qchain-portal",
      "request": "launch",
      "type": "dart",
      "deviceId": "chrome",
      "toolArgs": [
        "--dart-define=API_BASE_URL=https://<your-tunnel-name>.trycloudflare.com"
      ]
    },
    {
      "name": "QWallet (Local)",
      "cwd": "qchain-wallet",
      "request": "launch",
      "type": "dart",
      "deviceId": "<your-device-id>"
    },
    {
      "name": "QWallet (Cloudflare)",
      "cwd": "qchain-wallet",
      "request": "launch",
      "type": "dart",
      "deviceId": "<your-device-id>",
      "toolArgs": [
        "--dart-define=API_BASE_URL=https://<your-tunnel-name>.trycloudflare.com"
      ]
    }
  ]
}
```

---

### Method B: The public deployment

QPortal is no longer built into the Docker gateway. It is built and hosted by Vercel from the qchain-portal
repository (setup in its README), and the gateway's root address redirects there. QWallet is built locally as a
mobile app (`flutter build apk` / `flutter build ipa`).

---

## 10. Continuous Deployment (Self-Hosted Runner)

Once the stack above is running, pushes to `main` of **qchain-backend** redeploy the backend and the gateway
automatically. Two GitHub Actions workflows in its `.github/workflows/` do this, run by a self-hosted runner on the VM:

| Workflow | Triggered by changes to | Does |
|---|---|---|
| `deploy-backend.yml` | Go code, `go.mod`/`go.sum`, `docker/`, `.dockerignore` | `go test` (builder stage) → `docker/docker-build.sh` → `docker/docker-run.sh` → `/health` |
| `deploy-gateway.yml` | `gateway/` (except `*.md`) | `gateway/docker-build.sh` → `gateway/docker-run.sh` → `/gw-health` + the `/` redirect |

How they behave:

- **They deploy from the VM's own checkout** at `~/Desktop/QChain/teamqchain/qchain-backend`, running
  `git pull --ff-only` there first. They don't use a separate runner checkout, because the backend
  needs gitignored runtime files that only exist there (`.env`, and `crypto-material/` and `wallet/`
  in the `qchain-network` clone next to it).
- A deploy is refused if that checkout isn't on `main`, or if its tracked files were edited by hand.
- **Rollback:** before each build, the running image is kept as `:previous`. If the new container
  fails its health check, `:previous` is restarted and the run is marked failed.
- Chaincode, the MySQL schema and QWallet aren't covered. Deploy those by hand. QPortal deploys through Vercel.

**One-time runner setup** (needs admin on the GitHub repo):

1. **Lock down fork PRs first.** The repo is public, and the runner can read `.env` and use
   Docker. Go to *Settings → Actions → General → Approval for running fork pull request workflows
   from contributors*, choose **Require approval for all external contributors**, then **Save**.
2. Check the runner user is in the `docker` group (`id -nG`), and note the CPU architecture
   (`uname -m`).
3. On GitHub, in the **qchain-backend** repository, go to *Settings → Actions → Runners → New self-hosted runner*, then choose **Linux**
   and the matching architecture. On the VM, run that page's **Download** commands inside
   `~/actions-runner`. Then configure the runner. The token comes from the same page; it is
   single-use and expires in an hour.
   ```bash
   cd ~/actions-runner
   ./config.sh --url https://github.com/teamqchain/qchain-backend \
     --token <token-from-the-page> --name qchain-vm --labels qchain-vm --unattended
   ```
4. Install the runner as a service that starts on boot:
   ```bash
   sudo ./svc.sh install $USER
   sudo ./svc.sh start
   sudo ./svc.sh status    # → active (running); the Runners page now shows qchain-vm as Idle
   ```

**Removing the runner:** run `sudo ./svc.sh stop && sudo ./svc.sh uninstall`, then
`./config.sh remove --token <removal token from the Runners page>`.
