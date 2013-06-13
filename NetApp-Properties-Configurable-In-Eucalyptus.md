<TABLE border="1"
          summary="This table lists the NetApp properties exposed by Eucalyptus, a brief description, default value in Eucalyptus and NetApp CLI to verify the values">
<TR><TH>Release<TH>Eucalyptus property<TH>Allowed values<TH>Default value<TH>Description<TH>NetApp CLI
<TR><TH rowspan="15">3.2.1<TH colspan="5"> Applicable to both 7-mode and Cluster mode
<TR><TD>&lt;region>.storage.fractionalreserve<TD>Integer: 0-100<TD>0<TD><TD rowspan="4">vol options &lt;vol-name>
<TR><TD>&lt;region>.storage.guarantee<TD>String: “none”, “file”, “volume”<TD>“volume”<TD>
<TR><TD>&lt;region>.storage.noatimeupdate<TD>String: “on”, “off”<TD>“on”<TD>
<TR><TD>&lt;region>.storage.tryfirst<TD>String: “volume_grow”, “snap_delete”<TD>“volume_grow”<TD>
<TR><TD>&lt;region>.storage.enableautosize<TD>String: “true”, “false”<TD>“true”<TD><TD rowspan="3">vol autosize &lt;vol-name>
<TR><TD>&lt;region>.storage.volautosizemaxmultiplier<TD>Integer >= 1<TD>3<TD>
<TR><TD>&lt;region>.storage.volautosizeincrementinmb<TD>Integer >= 1<TD>256<TD>
<TR><TD>&lt;region>.storage.snappercent<TD>Integer >= 0<TD>0<TD><TD>snap reserve &lt;vol-name>
<TR><TH colspan="5"> 7-mode specific
<TR><TD>&lt;region>.storage.convertucode<TD>String: “on”, “off”<TD>“on”<TD><TD rowspan="2">vol options &lt;vol-name>
<TR><TD>&lt;region>.storage.createucode<TD>String: “on”, “off”<TD>“on”<TD>
<TR><TD>&lt;region>.storage.snapschedweeks<TD>Integer >= 0<TD>0<TD><TD rowspan="3">snap sched &lt;vol-name>
<TR><TD>&lt;region>.storage.snapscheddays<TD>Integer >= 0<TD>0<TD>
<TR><TD>&lt;region>.storage.snapschedhours<TD>Integer >= 0<TD>0<TD>
<TR><TD>&lt;region>.storage.aggregate<TD>String: CSV list of aggregates<TD><TD><TD>
<TR><TD>&lt;region>.storage.uselargestaggregate<TD>String: “true”, “false”<TD>“true”<TD><TD>

</TABLE>