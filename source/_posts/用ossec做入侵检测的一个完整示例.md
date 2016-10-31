---
title: 用ossec做入侵检测的一个完整示例
date: 2016-10-26 23:41:46
tags: 安全
---
...........
<!--more-->

```
 sinoiot@sinoiot-172-16-250-3:/var/ossec# ./bin/ossec-logtest -v
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/active-response_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/aix-ipsec_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/apache_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/arpwatch_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/asterisk_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/auditd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/checkpoint_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/cimserver_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/cisco-ios_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/cisco-vpn_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/clamav_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/courier_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/dovecot_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/dragon-nids_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/dropbear_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/ftpd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/grandstream_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/horde_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/imapd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/kernel-iptables_apparmor_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/mailscanner_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/mysql_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/named_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/netscreen_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/nginx_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/ntpd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/openbsd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/openldap_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/ossec_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/pam_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/pix_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/portsentry_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/postfix_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/postgresql_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/proftpd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/pure-ftpd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/racoon_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/roundcube_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/rshd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/samba_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/sendmail_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/snort_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/solares-bsm-audit_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/solaris-ipfilter_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/sonicwall_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/squid_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/ssh_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/su_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/sudo_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/suhosin_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/symantec_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/telnet_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/trend-osce_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/unbound_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/unix_chkpwd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/vm-pop3_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/vmware_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/vpopmail_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/vsftpd_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/web-accesslog_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/windows_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/wordpress_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/ossec_decoders/zeus_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/wazuh_decoders/amazon_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/wazuh_decoders/netscaler_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/wazuh_decoders/ossec_ruleset_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/wazuh_decoders/puppet_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/wazuh_decoders/redis_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Reading decoder file etc/wazuh_decoders/serv-u_decoders.xml.
2016/10/27 16:48:29 ossec-testrule: INFO: Started (pid: 19972).
ossec-testrule: Type one log per line.

Oct 27 16:20:01 shifudaotest CRON[11211]: pam_unix(cron:session): session opened for user root by (uid=0)
Oct 27 16:20:01 shifudaotest CRON[11211]: pam_unix(cron:session): session closed for user root
Oct 27 16:20:02 shifudaotest CRON[11210]: pam_unix(cron:session): session closed for user root

**Phase 1: Completed pre-decoding.
       full event: 'Oct 27 16:20:01 shifudaotest CRON[11211]: pam_unix(cron:session): session opened for user root by (uid=0)'
       hostname: 'shifudaotest'
       program_name: 'CRON'
       log: 'pam_unix(cron:session): session opened for user root by (uid=0)'

**Phase 2: Completed decoding.
       decoder: 'pam'

**Rule debugging:
    Trying rule: 1 - Generic template for all syslog rules.
       *Rule 1 matched.
       *Trying child rules.
    Trying rule: 5500 - Grouping of the pam_unix rules.
       *Rule 5500 matched.
       *Trying child rules.
    Trying rule: 5552 - PAM and gdm are not playing nicely.
    Trying rule: 5503 - User login failed.
    Trying rule: 5504 - Attempt to login with an invalid user.
    Trying rule: 5501 - Login session opened.
       *Rule 5501 matched.
       *Trying child rules.
    Trying rule: 5521 - Ignoring Annoying Ubuntu/debian cron login events.
       *Rule 5521 matched.

**Phase 3: Completed filtering (rules).
       Rule id: '5521'
       Level: '0'
       Description: 'Ignoring Annoying Ubuntu/debian cron login events.'
```
