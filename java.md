# Install

1. Download tarball and untar.
2. Move to install location, e.g. `$LOCAL/apps/` and set up any relevant sym links. 
3. Update `$PATH`.
4. Set relevant env vars, e.g.:
  ```
  export JAVA_HOME=$LOCAL/apps/java
  export JDK_HOME=$JAVA_HOME
  export JRE_HOME=$JAVA_HOME/jre
  export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
  ```
