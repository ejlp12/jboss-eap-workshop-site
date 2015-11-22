Start CLI dalam mode interaktif
```
./jboss-cli.sh -c 
```

Start ke host lain dan atau port yang berbeda
```
./jboss-cli.sh -c HOST:PORT --user=USERNAME [--password=PASSWORD]
```

Restart:
```
./jboss-cli.sh -c ":reload"
./jboss-cli.sh -c ":shutdown(restart=true)"
```

Versi JBoss EAP:
```
./jboss-cli.sh -c version
```

Daftar semua aplikasi yang di-deploy
```
./jboss-cli.sh -c deploy
./jboss-cli.sh -c "deploy -l"
```

Mendeploy aplikasi 
```
./jboss-cli.sh -c "deploy $HOME/tmp/hello.war"
```

Koneksi ke Domain Controller:
```
./jboss-cli.sh -c --controller=HOST:PORT
```

Menjalankan script file:
```
./jboss-cli.sh -c --file=/path/to/script.cli
```
