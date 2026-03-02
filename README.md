# cursive_ssh

`cursive_ssh` helps you painlessly turn a [cursive](https://crates.io/crates/cursive)-based terminal UI (TUI) into an application accessible over ssh. Designed to make the creation of BBS systems or ssh-based games simple, `cursive_ssh` takes a minimally opinionated approach to opening a TUI up to remote connections, beyond requiring you to use `cursive`. The ssh server implementation is provided by [russh](https://crates.io/crates/russh). The crate is a fork of the now archived [ssh_ui](https://crates.io/crates/ssh_ui) crate. This would not be possible without it's author, Ellen Poe.

The `main` function of the simplest `cursive_ssh`-based application looks something like this:

```
#[tokio::main]
async fn main() {
    let key_pair = KeyPair::generate_rsa(3072, SignatureHash::SHA2_256).unwrap();
    let mut server = AppServer::new_with_port(2222);
    let app = DialogApp {};
    server.run(&[key_pair], Arc::new(app)).await.unwrap();
}
```

First this generates a new keypair (but you should load several from disk for user-facing installations). Then it initializes a new `AppServer` on port 2222 and a new instance of a `DialogApp`, then calls `AppServer::run` to listen on the specified port for incoming connections. Let's look next at what makes `AppServer` tick.

```
struct DialogApp {}

impl App for DialogApp {
    fn on_load(&mut self) -> Result<(), Box<dyn Error>> {
        Ok(())
    }

    fn new_session(&self) -> Box<dyn AppSession> {
        Box::new(DialogAppSession::new())
    }
}
```

All it's doing here is providing a new `DialogAppSession` whenever there's a new incoming ssh connection. `DialogAppSession` is implemented as follows:

```
struct DialogAppSession {}

impl DialogAppSession {
    pub fn new() -> Self {
        Self {}
    }
}

impl AppSession for DialogAppSession {
    fn on_start(
        &mut self,
        _siv: &mut Cursive,
        _session_handle: SessionHandle,
        _pub_key: PublicKey,
        _force_refresh_sender: Sender<()>,
    ) -> Result<Box<dyn cursive::View>, Box<dyn Error>> {
        println!("on_start");
        Ok(Box::new(
            Dialog::around(TextView::new("Hello over ssh!"))
                .title("cursive_ssh")
                .button("Quit", |s| s.quit()),
        ))
    }
}
```

This is where the actual `cursive` TUI is created and returned to `cursive_ssh`. You can return whatever TUI you want, and `cursive_ssh` will take care of serving it to the client.

## Contributions

If you'd like to use `cursive_ssh` and it doesn't quite fit your needs, feel free to open an issue or pull request on the [GitHub repository](https://github.com/pignated/cursive_ssh).

## Thanks

I'd like to extend a special thanks to Ellen Poe, author of the ssh_ui crate, from which this is forked. 
