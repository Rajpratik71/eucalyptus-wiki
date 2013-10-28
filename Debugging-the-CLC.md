## Taking a heap dump on a live system
Use the following script to have the JVM take a heap dump (.hprof) and place it in /tmp/:
```
import javax.management.MBeanServer;
import java.lang.management.ManagementFactory;
import com.sun.management.HotSpotDiagnosticMXBean;

long timestamp = System.currentTimeMillis( )
String dumpfile = "/tmp/dump-${timestamp}.hprof"

MBeanServer server = ManagementFactory.getPlatformMBeanServer();
HotSpotDiagnosticMXBean bean = ManagementFactory.newPlatformMXBeanProxy(
    server,
    "com.sun.management:type=HotSpotDiagnostic",
    HotSpotDiagnosticMXBean.class);
bean.dumpHeap( dumpfile, false )
dumpfile
```

## Using Jprofiler 6
### Monitoring a live system
- Run through the Jprofiler Integration Wizard for a Remote Generic Application (this will allow you to download a tarball with the required libraries needed on your Eucalyptus component machine).
- Transfer that tarball to your machine and extract it in the /opt/jprofiler folder
- Add the following options to your CLOUD_OPTS:
```
--profile --profiler-home=/opt/jprofiler
```
- Place the jprofiler6 tarball/agent into /opt/jprofiler and extract it
- Restart the cloud process ```service eucalyptus-cloud restart```
- Connect to the JVM using the default port and passing your components IP

### Importing an HPROF file
- Go to the "Session" menu
- Click "Open Snapshot"
- Find your hprof file on your local machine 