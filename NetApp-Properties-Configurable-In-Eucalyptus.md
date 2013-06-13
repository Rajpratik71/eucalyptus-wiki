<TABLE border="1"
          summary="This table lists the NetApp properties exposed by Eucalyptus, a brief description, default value in Eucalyptus and NetApp CLI to verify the values">
<TR><TH>Release<TH>Eucalyptus property<TH>Allowed values<TH>Default value<TH>Description<TH>NetApp CLI
<TR><TH rowspan="8">3.2.1<TH colspan="5"> Applicable to both 7-mode and Cluster mode
<TR><TD>&lt;region>.storage.fractionalreserve<TD>Integer: 0-100<TD>0<TD><TD rowspan="4">vol options &lt;vol-name>
<TR><TD>&lt;region>.storage.guarantee<TD>String: “none” or “file” or “volume”<TD>“volume”<TD>
<TR><TD>&lt;region>.storage.noatimeupdate<TD>String: “on” or “off”<TD>“on”<TD>
<TR><TD>&lt;region>.storage.tryfirst<TD>String: “volume_grow” or “snap_delete”<TD>“volume_grow”<TD>
<TR><TD>&lt;region>.storage.enableautosize<TD>String: “true” or “false”<TD>“true”<TD><TD rowspan="3">vol autosize &lt;vol-name>
<TR><TD>&lt;region>.storage.volautosizemaxmultiplier<TD>Integer >= 1<TD>3<TD>
<TR><TD>&lt;region>.storage.volautosizeincrementinmb<TD>Integer >= 1<TD>256<TD>
</TABLE>