Hiera-based Puppet ENC
======================

This project acts as a miniature Puppet ENC in YAML files, made
accessible by Hiera. This makes server-side node configuration
(e.g. environments, custom node parameters) easy to edit in the same
way (and same place) as normal Hiera-based Puppet data.

Since an exec-based ENC simply prints out YAML when passed a node name,
the script just gets the merged hierarchical data with Hiera and then
prints.

Usage
-----

Deploy this repository on the puppetmaster as `/etc/puppet/hiera-enc`,
and add the following lines to `puppet.conf`:

    [master]
        node_terminus = exec
        external_nodes = /etc/puppet/hiera-enc/enc

Node configuration may be made in
`/etc/puppet/hiera-enc/nodes/<fqdn>.yaml`, or defaults set in the
`default.yaml` file.

Check the data returned for a given node by executing `./enc <fqdn>`,
just as Puppet itself will do. Note that the `DEBUG` lines are printed 
to STDERR and are not parsed by Puppet.

Example
-------

Assume the yaml files `hostname.example.com.yaml` and `default.yaml`:

    ---
    parameters:
      role: workstation
<!-- -->
    ---
    environment: production

When the enc is queried for `hostname.example.com`, the result is the
merged hash of these values:

    $ ./enc hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Hiera YAML backend starting
    DEBUG: 2014-07-11 13:04:24 +0200: Looking up parameters in YAML backend
    DEBUG: 2014-07-11 13:04:24 +0200: Looking for data source hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Found parameters in hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Looking up environment in YAML backend
    DEBUG: 2014-07-11 13:04:24 +0200: Looking for data source hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Looking for data source default
    DEBUG: 2014-07-11 13:04:24 +0200: Found environment in default
    ---
    parameters:
      role: workstation
    environment: production

If many nodes are sharing the same role (or environment) it may be
convenient to symlink node names to shared definition files. For
example, create a `workstation.def` and symlink workstation node fqdns
to this file, rather than copying the content repeatedly. A long
directory listing will therefore show:

    # ls -la nodes/
    total 12
    drwxr-xr-x 2 root root 4096 Jul 11 13:20 .
    drwxr-xr-x 6 root root 4096 Jul 11 13:20 ..
    lrwxrwxrwx 1 root root   15 Jul 11 13:20 hostname.example.com.yaml -> workstation.def
    -rw-r--r-- 1 root root   36 Jul 11 13:20 workstation.def

