## Taking a heap dump on a live system
Use the following script to have the JVM take a heap dump (.hprof) and place it in /tmp/:
```
import java.lang.management.ManagementFactory
import javax.management.JMX
import javax.management.MBeanServer
import javax.management.ObjectName
import javax.management.remote.JMXConnector
import javax.management.remote.JMXConnectorFactory
import javax.management.remote.JMXServiceURL
import com.eucalyptus.util.Exceptions
import com.sun.management.HotSpotDiagnosticMXBean

String url = "service:jmx:rmi:///jndi/rmi://127.0.0.1:1099/eucalyptus";
JMXServiceURL jmxURL = new JMXServiceURL(url);
JMXConnector connector = JMXConnectorFactory.connect(jmxURL);
ObjectName name = new ObjectName("com.sun.management:type=HotSpotDiagnostic");
String fname = "/tmp/${System.currentTimeMillis()}.hprof";
MBeanServer server = ManagementFactory.getPlatformMBeanServer( );
try {
  f = new File( fname );
  JMX.newMBeanProxy( server, name, HotSpotDiagnosticMXBean.class ).dumpHeap( fname, false );
  return "SUCCESS: Dumped heap to: ${fname} ${f.length()/(1024.0d*1024.0d)}MB";
} catch ( Exception ex ) {
  return Exceptions.string( ex );
}
```

## Using Jprofiler 6
### Monitoring a live system
- Add the following options to your CLOUD_OPTS:
```
--profile --profiler-home=/opt/jprofiler6
```
- Place the jprofiler6 tarball/agent into /opt/jprofiler6 and extract it
- Restart the cloud process ```service eucalyptus-cloud restart```
- Connect to the JVM using the default port and passing your components IP

### Importing an HPROF file
- Go to the "Session" menu
- Click "Open Snapshot"
- Find your hprof file on your local machine 