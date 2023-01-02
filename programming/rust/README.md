# rust tips

## musl build with openssl

```shell
# openssl build/install
sudo apt-get install \
    curl \
    ca-certificates \
    build-essential \
    musl-tools
```

```shell
export OPENSSL_VERSION=3.0.7
sudo mkdir -p /usr/local/musl/include
cd /tmp \
curl -fLO "https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz"
tar xvzf "openssl-$OPENSSL_VERSION.tar.gz"
cd "openssl-$OPENSSL_VERSION"
./config -fPIE -fPIC no-shared no-async --prefix=/usr/local/musl --openssldir=/usr/local/musl/ssl
make -j$(nproc)
make install
rm -r /tmp/https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz
```

openssl 3.x.x系からlib64になる？

```shell
export OPENSSL_LIB_DIR=/usr/local/musl/lib64/
export OPENSSL_INCLUDE_DIR=/usr/local/musl/include
export OPENSSL_STATIC=true
export PKG_CONFIG_ALLOW_CROOSS=1
```


## Cargo

### Debug - override

```toml
[patch.crates-io]
async-task = { path = "./async-task" }
```

依存関係にあるファイルのDebugを行いたいときはpatchを使用する


### dependenciesの書き方

```toml
test = { git = "https://github.com/test/test.git", branch = "master"}
test = { git = "https://github.com/test/test.git", rev = "<hash>"}
test = { path = "./<file path>" }
```

https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html


## async-std

簡単なchannel

```rust
use async_std::channel::{bounded,unbounded,RecvError};
use async_std::task;
use std::panic;

#[async_std::main]
async fn main() {
    let (s, r) = bounded(50);
    let mut counter = 0;
    let handle1 = task::spawn(async move {
        //s.send(counter).await;
        loop {
            s.send(counter).await;
            println!("[+] send: {}", counter);
            counter += 1;

            if counter == 50 {
                println!("[*] OK, that's enough");
                break;
            }
        }
    });

    let handle2 = task::spawn(async move {
        loop {

            if r.is_empty() {
                continue
            } else {
                let reciver = r.recv().await;
                let reciver = match reciver {
                    Ok(recv) => recv,
                    Err(RecvError) => {
                        panic!("{:?}", RecvError);
                        //println!("[!] Task error: {:?}", RecvError);
                    },
                };
                println!("[-] recv: {:?}", reciver);
            }
        }
    });

    task::block_on(async {
        handle1.await;
        handle2.await;
    });
}

```