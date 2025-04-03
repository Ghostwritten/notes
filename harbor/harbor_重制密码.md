

```bash
$ docker ps | grep harbor-db
4fefdbb78fa1   goharbor/harbor-db:v2.11.1            "/docker-entrypoint.…"   6 minutes ago   Up About a minute (healthy)                                           harbor-db

$ docker exec -it 4fefdbb78fa1 /bin/bash
postgres [ / ]$ psql -U postgres -d registry
psql (15.8)
Type "help" for help.

registry=# update harbor_user set salt='', password='' where user_id = 1;
UPDATE 1
registry=# exit
postgres [ / ]$ exit
exit

$ cd harbor

$ docker-compose stop
[+] Stopping 9/9
 ✔ Container nginx              Stopped                                                                                                                                                                                           0.2s
 ✔ Container harbor-jobservice  Stopped                                                                                                                                                                                           0.2s
 ✔ Container registryctl        Stopped                                                                                                                                                                                           0.2s
 ✔ Container harbor-portal      Stopped                                                                                                                                                                                           0.1s
 ✔ Container harbor-core        Stopped                                                                                                                                                                                           0.1s
 ✔ Container harbor-db          Stopped                                                                                                                                                                                           0.1s
 ✔ Container registry           Stopped                                                                                                                                                                                           0.1s
 ✔ Container redis              Stopped                                                                                                                                                                                           0.5s
 ✔ Container harbor-log         Stopped                                                                                                                                                                                          10.1s

$ docker-compose start
[+] Running 9/9
 ✔ Container harbor-log         Started                                                                                                                                                                                           0.2s
 ✔ Container registry           Started                                                                                                                                                                                           0.4s
 ✔ Container harbor-portal      Started                                                                                                                                                                                           0.5s
 ✔ Container registryctl        Started                                                                                                                                                                                           0.4s
 ✔ Container redis              Started                                                                                                                                                                                           0.4s
 ✔ Container harbor-db          Started                                                                                                                                                                                           0.5s
 ✔ Container harbor-core        Started                                                                                                                                                                                           0.2s
 ✔ Container harbor-jobservice  Started                                                                                                                                                                                           0.3s
 ✔ Container nginx              Started                                                                                                                                                                                           0.3s

```
重制后，密码恢复默认密码`Harbor12345`
