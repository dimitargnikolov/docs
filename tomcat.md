# Install

1. Download tarball and untar.
2. Move to install location, e.g. `$LOCAL/apps/` and set up any relevant sym links. 
3. Update `$PATH`.
4. Set up `jsvc` so we can run tomcat as a service:
  ```
  $ cd $CATALINA_HOME/bin
  $ tar xvfz common-daemon-native.tar.gz
  $ cd common-daemon-x.x.xx-native-src/unix
  $ ./configure --prefix=$LOCAL
  $ make
  $ cp jsvc $CATALINA_HOME/bin
  ```
5. Set relevant env vars, e.g.:
  ```
  export CATALINA_HOME=$LOCAL/apps/tomcat
  export CATALINA_BASE=$CATALINA_HOME
  export CATALINA_PID=$CATALINA_HOME/bin/catalina.pid
  export JSVC=$CATALINA_HOME/bin/jsvc
  export TOMCAT_USER=dimitar
  ```

# Starting
```
$ $CATALINA_HOME/bin/daemon.sh start
```

# Monitor Logs
```
$ tail -f $CATALINA_HOME/logs/catalina-daemon.out
```

# Stopping
```
$ CATALINA_HOME/bin/daemon.sh stop
```
