# Advanced Configuration

Cloudbreak Deployer configuration is based on environment variables. Cloudbreak Deployer always opens a new bash subprocess **without inheriting environment variables**. Only the following environment variables _are_ inherited:

- `HOME`
- `DEBUG`
- `TRACE`
- `CBD_DEFAULT_PROFILE`
- all `DOCKER_XXX`

To set environment variables relevant for Cloudbreak Deployer, add them to a file called `Profile`.

To see all available environment variables with their default values, run:

```
cbd env show
```

The `Profile` file is **sourced**, so you can use the usual syntax to set configuration values:

```
export MY_VAR=some_value
export OTHER_VAR=another_value
```


## Environment Specific Profiles

Let’s say that you want to use a different version of Cloudbreak for **prod** and **qa** profile. Since the `Profile` file is sourced, you will have to create two environment specific configurations that can be sourced:  
- `Profile.prod`  
- `Profile.qa`

For example, to create and use a **prod** profile, you need to:

1. Create a file called `Profile.prod`
2. Write the environment-specific `export DOCKER_TAG_CLOUDBREAK=0.3.99` into `Profile.prod` to specify Docker image.
3. Set the environment variable: `CBD_DEFAULT_PROFILE=prod`

To use the `prod` specific profile once, set:
```
CBD_DEFAULT_PROFILE=prod cbd some_commands
```

To permanently use the  `prod` profile, set `export CBD_DEFAULT_PROFILE=prod` in your `.bash_profile`.

## Available Configurations

### Using Your Own SSL Certificate

By default Cloudbreak is available only via HTTPS and it generates its own self-signed certificate. For most use cases such as testing or staging instances this is secure enough. However, if you want to use your own trusted certificate, you must manually configure it on the Cloudbreak host by replacing the existing certificate with your own certificate.

#### Prerequisites

 *  Resolvable domain name for the Cloudbreak hosts' IP addresses
 *  Generated valid certificate for the domain

#### Steps

 1.  First copy your private key and certificate to the host
 2.  Log in to the host machine via ssh, usually `ssh cloudbreak@[IP-ADDRESS]`
 3.  Make sure the domain name is resolvable for the host: `host [HOST-NAME]`
 4.  Change the host name of the machine using steps specific for your operating system
 5.  Identify your Cloudbreak location, which usually is `/var/lib/cloudbreak-deployer` and navigate there
 6.  Edit your `Profile` by adding or replacing `PUBLIC_IP` with `export PUBLIC_IP=[HOST-NAME]` where `[HOST-NAME]` is your actual Cloudbreak host
 7.  Configure TLS details in your `Profile` by adding the following line `export CBD_TRAEFIK_TLS="[CERT-LOCATION],[PRIV-KEY-LOCATION]"`
 8.  Restart Cloudbreak using the `cbd restart` command

### SMTP

If you want to change SMTP parameters, add them your `Profile`.  

The default values of the SMTP parameters are:
```
export CLOUDBREAK_SMTP_SENDER_USERNAME=
export CLOUDBREAK_SMTP_SENDER_PASSWORD=
export CLOUDBREAK_SMTP_SENDER_HOST=
export CLOUDBREAK_SMTP_SENDER_PORT=25
export CLOUDBREAK_SMTP_SENDER_FROM=
export CLOUDBREAK_SMTP_AUTH=true
export CLOUDBREAK_SMTP_STARTTLS_ENABLE=true
export CLOUDBREAK_SMTP_TYPE=smtp
```

#### SMTPS

If your SMTP server uses SMTPS, you must set the protocol in your `Profile` to `smtps`:
```
export CLOUDBREAK_SMTP_TYPE=smtps
```

### Certificates

#### Trusted Certificates

If the certificate used by the SMTP server is self-signed or the Java's default trust store doesn't contain it, you can add it to the trust store by copying it to `certs/trusted` inside the Cloudbreak Deployer directory, and start (or restart) the Cloudbreak container (with `cbd start`). On startup, the Cloudbreak container  automatically imports the certificates in that directory to its trust store.

#### Cloudbreak Certificate

If you would like to replace Cloudbreak's self-signed certificate with your own certificate then copy your certificate and the related private key under cloudbreak-deployment directory:
```
certs/traefik/mydomain.com.crt
certs/traefik/mydomain.com.key
```

If the traefik directory does not exsist then create it. After you have coped the cert and private key file, then add the following variable into the Profile file of cbd and restart cbd.

```
export CBD_TRAEFIK_TLS="/certs/traefik/mydomain.com.crt,/certs/traefik/mydomain.com.key"
```

As a best practice we recommend to replace `mydomain.com` with the actual domain what you want to use, but make sure that the actual file names and the values in `CBD_TRAEFIK_TLS` are identical.  

*Note: password protected private key files can't be used by Cloudbreak*  


###Access from Custom Domains

Cloudbreak Deployer supports multitenancy and uses UAA as an identity provider. In UAA, multitenancy is managed through identity zones. An identity zone is accessed through a unique subdomain. For example, if the standard UAA responds to [https://uaa.10.244.0.34.xip.io](https://uaa.10.244.0.34.xip.io), a zone on this UAA can be accessed through a unique subdomain [https://testzone1.uaa.10.244.0.34.xip.io](https://testzone1.uaa.10.244.0.34.xip.io).

If you want to use a custom domain for your identity or deployment, add the `UAA_ZONE_DOMAIN` line to your `Profile`:
```
export UAA_ZONE_DOMAIN=my-subdomain.example.com
```

For example, in our hosted deployment, the `identity.sequenceiq.com` domain refers to our identity server; therefore, the `UAA_ZONE_DOMAIN` variable has to be set to that domain. This variable is necessary for UAA to identify which zone provider should handle the requests that arrive to that domain.


### Consul

Cloudbreak uses [Consul](http://consul.io) for DNS resolution. All Cloudbreak-related services are registered as **someservice.service.consul**.

Consul’s built-in DNS server is able to fall back on another DNS server.
This option is called `-recursor`. Clodbreak Deployer first tries to discover the DNS settings of the host by looking for **nameserver** entry in the `/etc/resolv.conf` file. If it finds one, consul will use it as a recursor. Otherwise, it will use **8.8.8.8** .

For a full list of available consul config options, see Consul [documentation](https://consul.io/docs/agent/options.html).

To pass any additional Consul configuration, define a `DOCKER_CONSUL_OPTIONS` in the `Profile` file.

### SSH Fingerprint Verification

Cloudbreak is able to verify the SSH fingerprints of the provisioned virtual machines. We disable this feature by default for AWS and GCP, because we have experienced issues caused by the fact that Ccoud providers do not always print the SSH fingerprint into the provisioned machines console output. The fingerprint validation feature can be turned on by configuring the `CB_AWS_HOSTKEY_VERIFY` and/or the `CB_GCP_HOSTKEY_VERIFY` variables in your `Profile`. For example:
```
export CB_AWS_HOSTKEY_VERIFY=true
export CB_GCP_HOSTKEY_VERIFY=true
```

## Provider Specific Configurations

### Azure Resource Manager Command

- **cbd azure configure-arm**

For more information, see Azure [documentation](/azure/#azure-application-setup-with-cloudbreak-deployer).
