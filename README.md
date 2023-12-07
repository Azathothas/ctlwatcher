# CTL Watcher
Monitor Certificate Transparency logs for domains matching regexes.

# Overview
This project uses [CaliDog's CertStream-Server](https://github.com/CaliDog/certstream-server/issues) to
subscribe to the public lists of new TLS sertificates being recorded in various [Certificate Transparency Logs](https://certificate.transparency.dev) (CTLs).

New domains are checked against a user-supplies list of regexes, outputting matches

# Installing
```bash
sudo curl -qfsSL "https://raw.githubusercontent.com/Azathothas/Toolpacks/main/x86_64/ctlwatcher" -o "/usr/local/bin/ctlwatcher" && sudo chmod +xwr "/usr/local/bin/ctlwatcher" || curl -qfsSL "https://raw.githubusercontent.com/Azathothas/Toolpacks/main/x86_64/ctlwatcher" -o "$HOME/bin/ctlwatcher" && chmod +xwr "$HOME/bin/ctlwatcher" 2>/dev/null

ctlwatcher -h
```

# Building
```bash
  pushd $(mktemp -d) && git clone --filter "blob:none" "https://github.com/Azathothas/ctlwatcher" && cd "./ctlwatcher"
  export TARGET="x86_64-unknown-linux-gnu" ; rustup target add "$TARGET" ; export RUSTFLAGS="-C target-feature=+crt-static"
  sed '/^\[profile\.release\]/,/^$/d' -i "./Cargo.toml" ; echo -e '\n[profile.release]\nstrip = true\nopt-level = "z"\nlto = true' >> "./Cargo.toml"
  cargo build --target "$TARGET" --release ; mv "./target/$TARGET/release/ctlwatcher" "$HOME/bin/ctlwatcher" ; popd
```

# Setup
## CertStream-Server
```bash
!# Install
sudo curl -qfsSL "https://raw.githubusercontent.com/Azathothas/Toolpacks/main/x86_64/certstream-server-go" -o "/usr/local/bin/certstream-server-go" && sudo chmod +xwr "/usr/local/bin/certstream-server-go" || curl -qfsSL "https://raw.githubusercontent.com/Azathothas/Toolpacks/main/x86_64/certstream-server-go" -o "$HOME/bin/certstream-server-go" && chmod +xwr "$HOME/bin/certstream-server-go" 2>/dev/null
certstream-server-go -h

!# Run
#Configure certstream-server-go
   #kill zombie server
    sudo pgrep -f "certstream-server-go" | xargs sudo kill -9 || pgrep -f "certstream-server-go" | xargs kill -9 2>/dev/null
    rm "/tmp/server_config.yaml" 2>/dev/null 
   #Get Latest Config
    wget "https://raw.githubusercontent.com/Azathothas/Arsenal/main/certstream/server_config.yaml" -O "/tmp/server_config.yaml"
#Start Server
 certstream-server-go -config "/tmp/server_config.yaml"
 #nohup certstream-server-go -config "/tmp/server_config.yaml" >/dev/null 2>&1 &
```

Instead of running your own server you could just point ctlwatcher to the official server at `wss://certstream.calidog.io/`,
but to save CaliDog's bandwith I reccomend you run your own.

## Create Regexes
Then create a file containing regexes to match, one per line, e.g.:
```
ftp
\.com$
[0-9]+apple
```

Regex matching is using [this library](https://docs.rs/regex/latest/regex), which has an implicit `.*` at the start and end of
every pattern, if the `$^` anchors are not used.

# Running
```bash
# Where 'regexes.txt' contains list of regexes to match
ctlwatcher --regex-file regexes.txt --url "ws://localhost:8888"

# Use official/managed server
ctlwatcher --regex-file regexes.txt --url 'wss://certstream.calidog.io'

# Help and more details
ctlwathcer --help
```

# Example output
```
\.com -> www.quincassa.com.mx
\.com -> sa-sourcing.com
ftp.*\.azure.com$ -> ftp.sbzuvpxggxcdcos.atlas.cloudapp.azure.com
git.*staging -> git.git.staging-api.ugolek-lounge.ru
```
