# Dockerfiles

A random collection of dockerfiles I am generating as I play.  Mostly used for my desktop as inspired by:
https://blog.jessfraz.com/post/docker-containers-on-the-desktop/

```
                           
                                                       ``````
                                                       ;;;;;;
                                                       ;;;;;;
                                                       ;;;;;;
                                                       ;;;;;;
                                                       ;;;;;;
                                                       ......
                                         :::::: :::::: ::::::
                                         :;;;;; ;;;;;; ;;;;;;                  :;
                                         :;;;;; ;;;;;; ;;;;;;                  ;;;
                                         :;;;;; ;;;;;; ;;;;;;                 .;;;;
                                         :;;;;; ;;;;;; ;;;;;;                 ;;;;;,
                                         :;;;;; ;;;;;; ;;;;;;                 ;;;;;;
                                                                              ;;;;;;`
                                  `;;;;;.:;;;;; ;;;;;; ;;;;;; ;;;;;;          ;;;;;;, `.,.
                                  `;;;;;.:;;;;; ;;;;;; ;;;;;; ;;;;;;          ,;;;;;;;;;;;;;,
                                  `;;;;;.:;;;;; ;;;;;; ;;;;;; ;;;;;;           ;;;;;;;;;;;;;:
                                  `;;;;;.:;;;;; ;;;;;; ;;;;;; ;;;;;;           :;;;;;;;;;;;;
                                  `;;;;;.:;;;;; ;;;;;; ;;;;;; ;;;;;;            ;;;;;;;;;;;
                                  `;;;;;.:;;;;; ;;;;;; ;;;;;; ;;;;;;          .;;;;;;;;;;,
                                                                           .:;;;;;;:;:.
                            ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
                            ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;`
                            ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
                            ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;`
                            ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
                            :;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
                            .;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;:
                             ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
                             ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
                             ,;;;;;;;;;;;;;;;;.: ;;;;;;;;;;;;;;;;;;;;;;;;;;;;`
                              ;;;;;;;;;;;;;;;;.;;;;;;;;;;;;;;;;;;;;;;;;;;;;;.
                              ,;;;;;;;;;;;;;;;.:`;;;;;;;;;;;;;;;;;;;;;;;;;;,
                               ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;.
                                ;;;;;;;;;:.  ;;;;;;;;;;;;;;;;;;;;;;;;;;;;`
                                .;`          ,;;;;;;;;;;;;;;;;;;;;;;;;;;
                                 .;,          ;;;;;;;;;;;;;;;;;;;;;;;;,
                                  `;;          ;;;;;;;;;;;;;;;;;;;;;:
                                    ;;,         ;;;;;;;;;;;;;;;;;;;
                                     `;;,        ,;;;;;;;;;;;;;;,
                                       `;;;:`      :;;;;;;;;;,
                                          `:;;;;;;;;;;;;:.
                           
```

## Cleanup
(from http://blog.yohanliyanage.com/2015/05/docker-clean-up-after-yourself)

#### Remove exited containers
```
docker rm -v $(docker ps -a -q -f status=exited)
```

#### Remove ‘dangling’ images
```
docker rmi $(docker images -f "dangling=true" -q)
```

#### Remove unnwanted volumes
```
docker run -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker:/var/lib/docker --rm martin/docker-cleanup-volumes
```

