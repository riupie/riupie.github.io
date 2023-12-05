---
title: Bash Linux
weight: 210
menu:
  notes:
    name: Bash
    identifier: notes-sysadmin-bash
    parent: notes-sysadmin
    weight: 10
---

<!-- Linux -->
{{< note title="Rsync" >}}

```bash
$ rsync -avzhe 'ssh -p [port_number]' -P [user@remote_ip]:/path/to/file/source /path/to/file/destination
```

{{< /note >}}

{{< note title="Scan IP on Network" >}}

```bash
$ nmap -sP [ip_address/prefix]
```

{{< /note >}}

{{< note title="Search Available Version of Spesific Package on Debian Based OS" >}}

```bash
$ apt list -a <package_name>
```

{{< /note >}}


{{< note title="Install Spesific Kubernetes Version" >}}

```bash
$ sudo apt update;sudo apt install -qy kubelet=1.15.5-00 kubectl=1.15.5-00 kubeadm=1.15.5-00
```

{{< /note >}}

{{< note title="Open Spesific Port on Centos 7" >}}

```bash
iptables -I INPUT 5 -p tcp -m state --state NEW -m tcp --dport 9100 -j ACCEPT
```

{{< /note >}}

{{< note title="Replace String Using Sed" >}}

```bash
sed -i 's/old-string/new-string/g'
```

{{< /note >}}

{{< note title="Find and copy files" >}}

```bash
find /var/cache/dnf -iname "*.rpm" -exec cp {} packages/ \;
```

{{< /note >}}


{{< note title="Encrypt secret envar value using Google KMS" >}}

```bash
echo -n "This is my secret" | gcloud kms encrypt --plaintext-file=- --ciphertext-file=- --location=global --keyring=mykeyring --key=myappkey | base64 -w 0
```

{{< /note >}}


{{< note title="Encrypt file using Google KMS" >}}

```bash
gcloud kms encrypt \
    --key myappkey \
    --keyring mykeyring \
    --location global  \
    --plaintext-file application.properties \
    --ciphertext-file application.properties.enc
```

{{< /note >}}


{{< note title="Check DNS record" >}}

```bash
$ dig +noall +answer google.com

google.com.		204	IN	A	74.125.24.102
google.com.		204	IN	A	74.125.24.138
google.com.		204	IN	A	74.125.24.139
google.com.		204	IN	A	74.125.24.101
google.com.		204	IN	A	74.125.24.100
google.com.		204	IN	A	74.125.24.113
```

{{< /note >}}

{{< note title="Check service port" >}}

```bash
$ getent services 53

domain                53/tcp
```

{{< /note >}}


{{< note title="Troubleshoot SELinux Issue on RHEL/CentOS" >}}

```bash
# 1. Find your error from journalctl or audit.log then get the audit ID.

# 2. Analyze
grep 1624284378.419:2066 /var/log/audit/audit.log |audit2why
```

{{< /note >}}

{{< note title="Replace multiple file on OSX" >}}

```bash
find /to/my/path -type f -name "*.yaml" -exec sed -i '' -e 's/halo.com/hai.id/g' {} \;
```

{{< /note >}}