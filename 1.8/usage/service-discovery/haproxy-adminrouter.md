---
post_title: Configuring HTTP Proxy in Front of Admin Router
nav_title: Configure HTTP Proxy
menu_order: 61
---

You can configure a secure communication channel from a custom hostname and port to DC/OS Admin Router by using an HTTP Proxy.
 
The HTTP Proxy must perform on-the-fly HTTP request and response header modification because DC/OS is not aware of the custom hostname and port that is being used by user agents to address the HTTP proxy.

These instructions provide a tested [HAProxy](http://www.haproxy.org/) configuration example that handles the named request/response rewriting. This example ensures that the communication between HAProxy and DC/OS Admin Router is TLS-encrypted.


1.  Install HAProxy [1.6.9](http://www.haproxy.org/#down).

1.  Create an HAProxy configuration for DC/OS. This example is for a DC/OS cluster on AWS. For more information on HAProxy configuration parameters, see the [documentation](https://cbonte.github.io/haproxy-dconv/configuration-1.6.html#3).

    ```text
    global
      daemon
      log 127.0.0.1 local0
      log 127.0.0.1 local1 notice
      maxconn 20000
      pidfile /var/run/haproxy.pid
    defaults
      log            global
      option         dontlog-normal
      mode		 http
      retries             3
      maxconn          20000
      timeout connect  5000
      timeout client  50000
      timeout server  50000
    
    frontend http
      # Bind on port 9090. HAProxy will listen on port 9090 on each 
      # available network for new HTTP connections.
      bind 0.0.0.0:9090
      # Name of backend configuration for DC/OS.
      default_backend dcos
    
      # Store request Host header temporarily in transaction scope
      # so that its value is accessible during response processing.
      # Note: RFC 7230 requires clients to send the Host header and
      # specifies it to contain both, host and port information.
      http-request set-var(txn.request_host_header) req.hdr(Host)
    
      # Overwrite Host header to 'dcoshost'. This makes the Location
      # header in DC/OS Admin Router upstream responses contain a
      # predictable hostname (nginx uses this header value when
      # constructing absolute redirect URLs). That value is used
      # in the response Location header rewrite logic (see regular
      # expression-based rewrite in the backend section below).
      http-request set-header Host dcoshost
    
    backend dcos
      # Option 1: use TLS-encrypted communication with DC/OS Admin Router and
      # perform server certificate verification (including hostname verification).
      # DC/OS users must configure Admin Router with a custom TLS server
      # certificate, see https://dcos.io/docs/1.8/administration/securing-your-cluster/.
      # TLS-encrypted communication is included with Enterprise DC/OS.
      #
      # Explanation for the parameters in the following `server` definition line:
      # 
      # 1.2.3.4:443
      #
      #   IP address and port that HAProxy uses to connect to DC/OS Admin
      #   Router. This needs to be adjusted to your setup.
      #   
      #
      # ssl verify required
      #
      #   Instruct HAProxy to use TLS, and to error out if server certificate
      #   verification fails.
      #
      # ca-file dcos-ca.crt
      #
      #   The local file `dcos-ca.crt` is expected to contain the CA certificate
      #   that Admin Router's certificate will be verified against. It must be
      #   retrieved out-of-band (on Mesosphere Enterprise DC/OS this can be
      #   obtained via https://dcoshost/ca/dcos-ca.crt)
      #
      # verifyhost frontend-xxx.eu-central-1.elb.amazonaws.com
      #
      #   When verifying the TLS certificate presented by DC/OS Admin Router,
      #   perform hostname verification using the hostname specified here
      #   (expect the server certificate to contain a DNSName SAN that is
      #   equivalent to the hostname defined here). The hostname shown here is
      #   just an example and needs to be adjusted to your setup.
    
      server dcos-1 1.2.3.4:443 ssl verify required ca-file dcos-ca.crt verifyhost frontend-xxx.eu-central-1.elb.amazonaws.com
    
      # Variant 2: use TLS-encrypted communication with DC/OS Admin Router, but do
      # not perform server certificate verification (warning: this is insecure, and
      # we hope that you know what you are doing).
    
      # server dcos-1 1.2.3.4:443 ssl verify none
      server dcos-1 52.57.1.99:443 check-ssl ssl verify required ca-file dcos-ca.crt verifyhost frontend-ElasticL-JW6HOE0W6MAT-1017800469.eu-central-1.elb.amazonaws.com
    
      # Rewrite response Location header if it contains an absolute URL
      # pointing to the 'dcoshost' host: replace 'dcoshost' with original
      # request Host header (containing hostname and port).
      http-response replace-header Location https?://dcoshost((/.*)?) "http://%[var(txn.request_host_header)]\1"
    ```

1.  Start HAProxy with these settings. 