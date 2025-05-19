**ssh-keygen** is a command-line tool used to generate SSH key pairs (public and private keys) for secure authentication. It creates two files:
- Private key (`id_rsa` or `id_ed25519`): You keep this secret.
- Public key (`id_rsa.pub` or `id_ed25519.pub`): You share this with the remote server.

SSH then uses asymmetric encryption to verify you without needing a password.
