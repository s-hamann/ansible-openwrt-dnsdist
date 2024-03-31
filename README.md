OpenWrt dnsdist
===============

This role configures [dnsdist](https://dnsdist.org/) on [OpenWrt](https://www.openwrt.org/) targets.

Requirements
------------

This role requires the `ansible.utils` collection and the [netaddr](https://github.com/netaddr/netaddr/) Python package on the Ansible controller.

Moreover, it requires a working [Python](https://www.python.org/) installation on the target system or [gekmihesg's Ansible library for OpenWrt](https://github.com/gekmihesg/ansible-openwrt) on the Ansible controller.

Role Variables
--------------

* `dnsdist_package`  
  The name of the dnsdist package to install.
  As of OpenWrt 23.05, `dnsdist` and `dnsdist-full` are available.
  Defaults to `dnsdist`, but setting certain role variables forces this to `dnsdist-full`.
* `dnsdist_console`  
  A list of IP addresses with an option port to listen on for connections to the console.
  This setting maps to the [`controlSocket`](https://dnsdist.org/reference/config.html#controlSocket) function.
* `dnsdist_console_key`  
  A Base64 encoded random 256 bit value to use as shared secret between dnsdist console client and server.
  This setting maps to the [`setKey`](https://dnsdist.org/reference/config.html#setKey) function.
  Defaults to a randomly generated key based on Ansible's `inventory_hostname`.
  It is highly recommended to set this to a securely generated and securely stored value instead.
* `dnsdist_console_acl`  
  A list of networks in CIDR notation that are allowed to connect to the console.
  This setting maps to the [`addConsoleACL`](https://dnsdist.org/reference/config.html#addConsoleACL) function.
* `dnsdist_listen`, `dnsdist_listen_dot`, `dnsdist_listen_doh`, `dnsdist_listen_dnscrypt`  
  A list of IP addresses with an optional port to listen on for unencrypted DNS (Do53, DoU), DNS-over-TLS (DoT), DNS-over-HTTPS (DoH) and DNSCrypt, respectively.
  These settings map to the functions [`addLocal`](https://dnsdist.org/reference/config.html#addLocal), [`addTLSLocal`](https://dnsdist.org/reference/config.html#addTLSLocal), [`addDOHLocal`](https://dnsdist.org/reference/config.html#addDOHLocal), [`addDNSCryptBind`](https://dnsdist.org/reference/dnscrypt.html#addDNSCryptBind), respectively.
  For `dnsdist_listen` and `dnsdist_listen_doh`, list items may be strings that specify the address.
  For all settings, list items may be dictionaries with the following keys:
  * `address`  
    The address to listen on with an optional port.
    Use `[]` around IPv6 addresses.
    The port must be separated from the IP address by a `:`.
    Default ports depend on the protocol.
    For DNSCrypt, the port is not optional.
    Mandatory.
  * `cert`  
    Path to a X.509 certificate in PEM format or a list of paths.
    Optional for DoH, ignored for Do53, otherwise mandatory.
  * `key`  
    Path to a private key file corresponding to the certificate or a list of paths.
    Optional for DoH, ignored for Do53, otherwise mandatory.
  * `urls`  
    Path part of a URL to accept DoH queries on or a list of paths.
    Optional for DoH, ignored otherwise.
  * `provider`  
    The provider name for this bind.
    Mandatory for DNSCrypt, ignored otherwise.
  * `options`  
    A dictionary of options to pass to the respective function.
    Valid keys and values depend on the protocol.
    Refer to the respective function documentation for valid options and their meaning.
    Options given here take precedence over those in `dnsdist_listen_default_options` and friends.
    Optional.
* `dnsdist_listen_default_options`, `dnsdist_listen_dot_default_options`, `dnsdist_listen_doh_default_options`, `dnsdist_listen_dnscrypt_default_options`  
  A dictionary of options to pass to the respective listeners.
  See `options` subkey of `dnsdist_listen` and friends.
  Optional.
* `dnsdist_acl`  
  A list of networks in CIDR notation that are allowed to send DNS queries.
  This setting maps to the [`addACL`](https://dnsdist.org/reference/config.html#newACL) function.
* `dnsdist_backend`  
  A list of backend servers.
  This setting maps to the [`addServer`](https://dnsdist.org/reference/config.html#newServer) function.
  Each list item is passed to `addServer` and may therefore be either a simple string in the format IP:PORT or a dictionary of settings for this backend.
  Refer to the function documentation for valid keys and their meaning.
* `dnsdist_cache`  
  A list of [cache](https://dnsdist.org/guides/cache.html) definitions.
  Each list item is a dictionary with the following keys:
  * `pool`  
    A list of pool names (defined in `dnsdist_backend`) for which to use this cache.
    Defaults to the unnamed default pool.
  * `maxEntries`  
    The maximum number of entries in this cache.
    Mandatory.
  * `options`  
    A dictionary of options to pass to [`newPacketCache`](https://dnsdist.org/reference/config.html#newPacketCache).
    Refer to the function documentation for valid options and their meaning.
    Optional.
* `dnsdist_rule`  
  A dictionary of [rule selectors](https://dnsdist.org/reference/selectors.html) to be referenced later in `dnsdist_action`.
  Rules defined here do not do anything by themselves.
  Dictionary keys are references to the rules for later use and values are the actual rule selector definitions.
  Dictionary keys must be valid Lua variable names (i.e. any string of letters, digits, and underscores, not beginning with a digit).
  Rule selector definitions are either:
  1. A dictionary with *a single* key value pair.  
    The key is the rule selector name (e.g. `AllRule`), with or without the `Rule` suffix.
    The value is the option or list of options to pass to the rule selector.
    For the rule selectors `AndRule`, `OrRule` and `NotRule`, the value is instead interpreted as a rule or list of rules.
  2. A simple string.  
    A key used previously in `dnsdist_rule`.
    This is really only useful for `AndRule`, `OrRule` and `NotRule` (see above) or in `dnsdist_action`.
* `dnsdist_action`  
  A list of rule action definitions.
  This setting maps to the [`addAction`](https://dnsdist.org/reference/rules-management.html#addAction) function.
  Each list item must be a dictionary with the following keys:
  * `rule`  
    A rule definition as described in `dnsdist_rule` or a reference to a rule defined in `dnsdist_rule`.
  * `action`  
    An [action](https://dnsdist.org/reference/actions.html) definition.
    Action definitions work similar to rule definitions and must be a dictionary with *a single* key value pair.  
    The key is the action name (e.g. `AllowAction`), with or without the `Action` suffix.
    The value is the option or list of options to pass to the action.
* `dnsdist_raw`  
  A list of raw strings to add to the configuration file.
  They may be single lines or multi-line strings.
  This setting can be used to customize the installation beyond what the aforementioned settings provide.

Dependencies
------------

This role does not depend on any specific roles.

Example Configuration
---------------------

The following is a short example for some of the configuration options this role provides:

```yaml
# Enable the console
dnsdist_console:
  - 127.0.0.1
# Set up Do53 listeners for IPv4 and IPv6
dnsdist_listen_default_options:
  reusePort: true
  numberOfStoredSessions: 0
dnsdist_listen:
  - 0.0.0.0
  - '[::]'
# Set up DoT listeners for IPv4 and IPv6
dnsdist_listen_dot_default_options:
  reusePort: true
dnsdist_listen_dot:
  - address: 0.0.0.0
    cert: /etc/ssl/dnsdist.pem
    key: /etc/ssl/dnsdist.key
  - address: '[::]'
    cert: /etc/ssl/dnsdist.pem
    key: /etc/ssl/dnsdist.key
# Set up backends
dnsdist_backend:
  # local recursive resolver in the default pool
  - address: 127.0.0.1:5301
    name: 'local resolver'
  # local authoritative name server for the internal zone
  - address:  127.0.0.1:5302
    name: 'local authoritative dns'
    pool: auth
    useClientSubnet: true
  # some fallback recursive resolvers
  - address: 9.9.9.9
    pool: fallback
    name: quad9
    tls: openssl
    subjectName: dns.quad9.net
  - address: 2620:fe::fe
    pool: fallback
    name: quad9
    tls: openssl
    subjectName: dns.quad9.net
# Set up caches
dnsdist_cache:
  # Cache authoritative name server
  - pool:
      - auth
    options:
      keepStaleData: true
      parseECS: true
  # Separate cache for recursive resolvers
  - pool:
      - ''
      - fallback
dnsdist_rule:
  # Match clients from the internal network
  internal_network:
    NetmaskGroup:
      - 127.0.0.1/8
      - ::1/128
      - 192.168.0.0/16
  # Match queries for the internal zones
  internal_zone:
    QNameSuffix:
      - example.internal
      - 168.192.in-addr.arpa
dnsdist_action:
  # Direct internal clients' queries for the internal zone to the authoritative name server pool
  - rule:
      And:
        - internal_network
        - internal_zone
    action:
      Pool: auth
  # If the internal resolver is down, use the fallback pool
  - rule:
      Not:
        PoolAvailable: ''
    action:
      Pool: fallback
# Raw settings not specifically handled by this role
dnsdist_raw:
  - 'setECSOverride(true)'
  - 'setECSSourcePrefixV4(24)'
  - 'setECSSourcePrefixV6(64)'
```

License
-------

MIT
