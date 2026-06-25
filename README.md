# Setup SSH

A composite Gitea/GitHub Action that writes SSH keys from secrets into `~/.ssh`
and scans the host key, so later steps can run `ssh`/`scp`/`rsync` against a
remote server without interactive prompts.

## Usage

```yaml
# Setup the SSH
-
    name: Setup the SSH
    uses: AnggitHerdianto/action-setup-ssh@main
    with:
        host: example.com
        id_ecdsa: ${{ secrets.ID_ECDSA }}
        id_ecdsa_pub: ${{ secrets.ID_ECDSA_PUB }}
```

After this step, subsequent steps can connect directly:

```yaml
-
    name: Deploy
    run: ssh deploy@example.com "cd /var/www && git pull"
```

## Inputs

| Name             | Required | Default | Description                          |
| ---------------- | -------- | ------- | ------------------------------------ |
| `host`           | yes      | —       | SSH host address or IP to connect to |
| `port`           | no       | `22`    | SSH port number                      |
| `id_ecdsa`       | no\*     | —       | Private ECDSA key content            |
| `id_ecdsa_pub`   | no       | —       | Public ECDSA key content             |
| `id_ed25519`     | no\*     | —       | Private Ed25519 key content          |
| `id_ed25519_pub` | no       | —       | Public Ed25519 key content           |
| `id_rsa`         | no\*     | —       | Private RSA key content              |
| `id_rsa_pub`     | no       | —       | Public RSA key content               |

\* At least one private key must be provided. Each key is only written when its
value is not empty, so you can supply just the key type you use.

## Outputs

| Name     | Description                            |
| -------- | -------------------------------------- |
| `stdout` | Standard output of executed commands. |

## Important notes

- **Store keys as secrets, never in plain text.** Put the key contents in your
  repository/organization secrets (e.g. `ID_ECDSA`) and reference them with
  `${{ secrets.ID_ECDSA }}`.

- **The private key must be complete and intact**, including the
  `-----BEGIN ... -----` / `-----END ... -----` lines and the original line
  breaks. Pasting it as a single line will corrupt the key and authentication
  will fail. Copy the file as-is (e.g. `cat ~/.ssh/id_ecdsa | pbcopy`).

- **The matching public key must already be in the server's**
  `~/.ssh/authorized_keys`. This action only configures the runner side — if the
  public key is not trusted on the server, the connection is still rejected.

- **The public key inputs are optional** for connecting. Only the private key is
  needed by the runner; provide the `*_pub` inputs only if a later step needs the
  public key file.

- **Provide only the key type you use.** Empty inputs are skipped, so passing
  just `id_ecdsa` (without `id_rsa` / `id_ed25519`) works fine.

- **Non-default SSH port:** set `port` so the host key scan and connections use
  the right port:

  ```yaml
  with:
      host: example.com
      port: 2222
      id_ecdsa: ${{ secrets.ID_ECDSA }}
  ```

## License

Maintained by Anggit Herdianto.
