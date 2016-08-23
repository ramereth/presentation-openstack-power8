Building and running OpenStack on POWER8
========================================

.. revealjs:: Building and running OpenStack on POWER8
  :title-heading: h2
  :subtitle: Lance Albertson
  :subtitle-heading: h3

  Oregon State University Open Source Lab

  @ramereth

.. revealjs:: Summary

  .. rst-class:: fragment

  .. raw:: html

    <ul>
    <li class="fragment">POWER8 Overview</li>
    <li class="fragment">POWER at OSUOSL</li>
    <li class="fragment">Building a RHEL-based P8 platform with Openstack</li>
    <li class="fragment">Architecture porting issues</li>
    <li class="fragment">Problems we have encountered with OpenStack</li>
    <li class="fragment">Openstack deployment with Chef</li>
    <li class="fragment">OSL Wrapper cookbook</li>
    <li class="fragment">Next Steps</li>
    </ul>

.. revealjs:: Disclaimer

.. revealjs:: POWER8 Overview
  :title-heading: h5

  .. revealjs:: Design

    - Designed to be a massively multithreaded chip
    - Designs are available for licensing under the OpenPOWER Foundation
    - Little-Endian & Big-Endian
    - Several non-IBM companies building P8 hardware

      - Tyan, Rackspace (OpenCompute-based) & Google

  .. revealjs:: OpenPower Abstraction Layer (OPAL)

    - OPAL is the new Open Source firmware for POWER8
    - Acts as an on-system HMC
    - Enables the machine to boot similar to PC servers
    - Linux Kernel and loads the boot loader Petitboot
    - Petitboot provides a shell environment for debugging and setup
    - Petitboot will use kexec and boot into the system kernel

.. revealjs:: POWER at OSUOSL
  :title-heading: h5

  .. revealjs:: History

    - Providing PPC64 compute resources since 2005
    - Close collaboration with IBM LTC
    - POWER5, POWER7 and now POWER8
    - OSL managed LPAR deployment to make it easier on projects
    - Pre-P8 Projects:

      - Debian, Gentoo, Fedora, PostgreSQL
      - Linux Foundation, Haskell, GoLang
      - Mozilla, OpenSUSE, LLVM, GCC

  .. revealjs:: POWER8 at OSUOSL

    - Goal is to provide on-demand PPC64/PPC64LE compute resources to FOSS projects
    - Assist with ppc64/ppc64le porting & testing
    - Expose OSU students to OpenStack and POWER8
    - Collaboration with IBM engineers on architecture issues
    - Create a vanilla Openstack cluster for FOSS projects

  .. revealjs:: Projects running on our P8 cluster

    - CloudFoundry, Docker, CentOS, CouchDB
    - Haskell, Glibc, JXcore, LLVM, NodeJS
    - OpenJDK, GoLang, oVirt, libjpeg-turbo
    - BLCR, Gentoo

.. revealjs:: Building a RHEL-based P8 platform with Openstack
  :title-heading: h5

  .. revealjs:: Supported OS platforms

    .. rst-class:: fragment

      PowerKVM

      Ubuntu

      RHEL

    .. rv_note::

      - PowerKVM is a Fedora based platform built by IBM for running OpenStack on POWER8
      - Ubuntu is fairly well supported community wide
      - RHEL is just starting to get support

  .. revealjs:: Decision to use RHEL

    - Little community support at the time and opportunity to help the community
    - We use CentOS internally as our primary OS & more familiar with the RHEL eco-system
    - RHEL has the RDO OpenStack distribution that is well supported
    - Chef support with OpenStack needed some help
    - I love challenges!

  .. revealjs:: OpenStack Architecture (old)

    - Started in 2014
    - Icehouse
    - Controller node

      - Runs all public API services, dashboard
      - DB hosted on a shared bare-metal system
      - X86_64 CentOS 6 VM running on Ganeti+KVM
    - Compute node(s)

      - Nova compute and networking
      - Flat networking
      - PPC64 Fedora

  .. revealjs:: OpenStack Architecture (new)

    - Deployed 2016 (deployed last week)
    - Mitaka
    - Controller node

      - Runs all public API services, dashboard
      - DB hosted on a shared bare-metal system
      - X86_64 CentOS 7 VM running on Ganeti+KVM
    - Compute node(s)

      - Nova Compute
      - Neutron Networking

        - Linuxbridge
        - Provider and Tenant networking using VXLAN
      - PPC64LE CentOS 7.2

  .. revealjs:: Compute nodes

    - Did initial development on Fedora 19
    - Fedora 20 PPC64 base system (old)
    - Fedora 21 versions of a few packages
    - CentOS 7.2 PPC64LE base system (new)

.. revealjs:: Architecture porting issues
  :title-heading: h5

  .. revealjs:: Chef

    - No PPC64/PPC64LE Chef client
    - Needed to build our own chef-client
    - Omnibus

      - Bootstrap build env
      - Build dependency issues
      - Architecture configuration issues in Omnibus
    - Chef has stable ppc64/ppc64le builds today

  .. revealjs:: Package Support

    - Support for P8 was bleeding edge and new features were added weekly
    - Built versions of latest packages from Fedora rawhide packages:

      - qemu
      - libvirt
      - kernel
    - Internal repo for these custom packages:

      - http://ftp.osuosl.org/pub/osl/repos/yum/openpower/centos-7/ppc64le/
    - Kernel required a few custom options to be enabled
    - Runtime setup: Disable SMT

    .. rv_note::

      - No hotplug support, added via a patch I got on a mailing list

  .. revealjs:: Guest OS images

    - Few OS supported ppc64/ppc64le or provided guest images pre-built
    - Variety of tools which are platform specific
    - Missing support for cloud-init
    - Initially started creating images manually with qemu directly

  .. revealjs:: Packer -- Multi platform support

    - We needed Go to use Packer
    - GoLang support was literally in the works
    - Finally built our own packer binary last Nov!

      - http://ftp.osuosl.org/pub/osl/openpower/rpms/
    - WIP Packer Templates:

      - https://github.com/osuosl/bento/tree/ramereth/ppc64

  .. revealjs:: Architecture issues

    - OPAL firmware bugs
    - pre-P8 machines were very buggy
    - IPMI console would sometimes stop working
    - Random lockups
    - Included HW RAID, but no cached write-back support

.. revealjs:: Problems we have encountered with OpenStack
  :title-heading: h5

  .. revealjs:: Learning and understanding OpenStack

    - Lots of moving pieces
    - Neutron networking is complex and a moving target
    - Deciding on the proper design architecture for our use case

  .. revealjs:: Bugs and "Features"

    - Interaction between libvirt and nova-compute was buggy at times
    - Some bugs were just Icehouse itself, others were architecture specific
    - Learning how to deploy Openstack and making (gasp) mistakes!
    - Iptables issues between Chef and Openstack
    - Provider networks configures dnsmasq as an open resolver
    - SSL API endpoints

  .. revealjs:: Stability

    - Rabbitmq would constantly need to be restarted
    - nova-compute services would randomly stop working
    - Running Fedora on compute and CentOS on controller made things ... interesting

.. revealjs:: CentOS 7 ppc64le on OpenStack
  :title-heading: h5

  .. revealjs:: RHEL / CentOS support

    - Introduced in 7.1 and fully supported in 7.2
    - CentOS community was still bootstrapping and testing
    - We built our own pre-release CentOS 7.2 for testing
    - Using ppc64le on compute nodes

  .. revealjs:: RDO

    - Community for deploying Openstack on CentOS, Fedora and RHEL
    - Repositories built against each Platform
    - Each release of OpenStack separated

  .. revealjs:: RHEV (Red Hat Enterprise Virtualization)

    - Updated KVM packaging
    - Part of the Virt SIG of CentOS
    - Used SRPMs to build ppc64le versions in a location repo
    - One patch needed to work around bug

.. revealjs:: Openstack Deployment with Chef
  :title-heading: h5

  .. revealjs:: Why Chef?

    - Primary CM tool used at the OSL
    - Provides a lot of testing capability on deployment
    - Can use the full power of the Ruby language for configuring the cluster

  .. revealjs:: Chef Openstack

    - Set of cookbooks that will deploy the various services of Openstack
    - Part of the OpenStack umbrella
    - Community driven
    - Did a major refactor of the code for Mitaka release

  .. revealjs:: OSL Openstack

    - Created a wrapper cookbook (osl-openstack)
    - https://github.com/osuosl-cookbooks/osl-openstack
    - OSL site specific configuration
    - Split recipes out by upstream cookbook name
    - Contains ppc64le specific changes
    - Currently only tested on CentOS 7

.. revealjs:: OSL Wrapper cookbook
  :title-heading: h5

  .. revealjs:: recipes/default.rb

    - `recipes/default.rb`__
    - Default configuration for cluster
    - Include local yum repos
    - Include command clients
    - Logic around endpoints

    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/recipes/default.rb

  .. revealjs:: recipes/identity.rb

    - `recipes/identity.rb`__
    - Just includes recipes
    - Some wrapper, some upstream
    - Allows us to test just Keystone by itself

    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/recipes/identity.rb

  .. revealjs:: recipes/controller.rb

    - `recipes/controller.rb`__
    - Pulls in all wrapper recipes needed to build a controller
    - Allows for us to split things out eventually if we want to

    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/recipes/controller.rb

  .. revealjs:: Testing and development

    - Unit Testing

      - ChefSpec
      - RSpec
    - Integration Testing

      - Test Kitchen
      - ServerSpec
    - Chef Provisioning

      - Deploy VMs as controller/compute
      - Deploy on bare-metal for a test cluster

  .. revealjs:: Unit Testing

    - Ensure the Chef code is doing what it's supposed to do
    - Easily test Architecture-specific logic
    - Verify configuration files contain proper settings
    - Examples:

      - `spec/default_spec.rb`__
      - `spec/compute_controller.rb`__
      - `spec/linuxbridge_spec.rb`__

    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/spec/default_spec.rb
    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/spec/compute_controller_spec.rb
    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/spec/linuxbridge_spec.rb

  .. revealjs:: Unit Testing (output)

    .. rv_code::

      $ rspec spec/default_spec.rb

      osl-openstack::default
        includes cookbook base::ifconfig
        includes cookbook selinux::permissive
        includes cookbook yum-qemu-ev
        includes cookbook openstack-common
        includes cookbook openstack-common::logging
        includes cookbook openstack-common::sysctl
        includes cookbook openstack-identity::openrc
        includes cookbook openstack-common::client
        includes cookbook openstack-telemetry::client
        setting arch to x86_64
          does not add OSL-Openpower repository on x86_64
        setting arch to ppc64
          add OSL-openpower-openstack repository on ppc64
        setting arch to ppc64le
          add OSL-openpower-openstack repository on ppc64le
        /etc/sysconfig/iptables-config
          should render file "/etc/sysconfig/iptables-config" with content /^IPTABLES_SAVE_ON_STOP="yes"$/
          should render file "/etc/sysconfig/iptables-config" with content /^IPTABLES_SAVE_ON_RESTART="yes"$/

      Finished in 16.88 seconds (files took 2.83 seconds to load)
      14 examples, 0 failures


      ChefSpec Coverage report generated...

        Total Resources:   1
        Touched Resources: 1
        Touch Coverage:    100.0%

      You are awesome and so is your test coverage! Have a fantastic day!

  .. revealjs:: Test Kitchen & ServerSpec

    - Test Kitchen

      - Test CLI tool which allows you to execute the configured code on one or more platforms
      - Integrates with testing frameworks
      - Must have tool for Chef users
      - Configured via `.kitchen.yml`__
    - ServerSpec

      - RSpec tests for configured servers
      - Integration tests
      - Ensures things actually happen on the system
      - Example: `test/integration/default/serverspec/default_spec.rb`__

    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/.kitchen.yml
    .. __: https://github.com/osuosl-cookbooks/osl-openstack/blob/master/test/integration/default/serverspec/default_spec.rb

  .. revealjs:: Test Kitchen (list)

    .. rv_code::

      $ kitchen list
      Instance                            Driver     Provisioner  Verifier  Transport  Last Action
      default-centos-72                   Openstack  ChefZero     Busser    Rsync      Not Created
      mon-centos-72                       Openstack  ChefSolo     Busser    Rsync      Not Created
      mon-controller-centos-72            Openstack  ChefSolo     Busser    Rsync      Not Created
      ops-messaging-centos-72             Openstack  ChefZero     Busser    Rsync      Not Created
      identity-centos-72                  Openstack  ChefZero     Busser    Rsync      Not Created
      image-centos-72                     Openstack  ChefZero     Busser    Rsync      Not Created
      network-centos-72                   Openstack  ChefZero     Busser    Rsync      Not Created
      linuxbridge-centos-72               Openstack  ChefZero     Busser    Rsync      Not Created
      compute-controller-centos-72        Openstack  ChefZero     Busser    Rsync      Not Created
      compute-centos-72                   Openstack  ChefZero     Busser    Rsync      Not Created
      dashboard-centos-72                 Openstack  ChefZero     Busser    Rsync      Not Created
      block-storage-centos-72             Openstack  ChefZero     Busser    Rsync      Not Created
      block-storage-controller-centos-72  Openstack  ChefZero     Busser    Rsync      Not Created
      telemetry-centos-72                 Openstack  ChefZero     Busser    Rsync      Not Created
      controller-centos-72                Openstack  ChefZero     Busser    Rsync      Not Created
      allinone-centos-72                  Openstack  ChefZero     Busser    Rsync      Not Created

  .. revealjs:: Test Kitchen (test)

    .. rv_code::

      $ kitchen test default
      -----> Starting Kitchen (v1.8.0)
      -----> Cleaning up any prior instances of <default-centos-72>
      -----> Destroying <default-centos-72>...
             Finished destroying <default-centos-72> (0m0.00s).
      -----> Testing <default-centos-72>
      -----> Creating <default-centos-72>...
             OpenStack instance with ID of <a25fa410-5caf-4f96-bddb-1e6daddd06d9> is ready.
      ...

      Chef Client finished, 115/198 resources updated in 03 minutes 12 seconds
      Finished converging <default-centos-72> (3m41.31s).
      -----> Setting up <default-centos-72>...
      Finished setting up <default-centos-72> (0m0.00s).
      -----> Verifying <default-centos-72>...
      Preparing files for transfer
      ...

      Yumrepo "RDO-mitaka"
        should exist
        should be enabled

      File "/root/openrc"
        content
          should match /
      export OS_USERNAME=admin
      export OS_PASSWORD=admin
      export OS_TENANT_NAME=admin
      export OS_AUTH_URL=http:\/\/.*:5000\/v2.0
      export OS_REGION_NAME=RegionOne/

      File "/root/openrc"
        content
          should not match /OS_AUTH_URL=http:\/\/127.0.0.1\/v2.0/

      File "/etc/sysconfig/iptables-config"
        content
          should match /^IPTABLES_SAVE_ON_STOP="yes"$/
        content
          should match /^IPTABLES_SAVE_ON_RESTART="yes"$/

      Finished in 1.64 seconds (files took 0.67905 seconds to load)
      6 examples, 0 failures

      Finished verifying <default-centos-72> (0m17.00s).
      -----> Destroying <default-centos-72>...
      OpenStack instance <a25fa410-5caf-4f96-bddb-1e6daddd06d9> destroyed.
      Finished destroying <default-centos-72> (0m1.84s).
      Finished testing <default-centos-72> (4m57.57s).
      -----> Kitchen is finished. (4m58.60s)

.. revealjs::  Next steps
  :title-heading: h5

  .. revealjs:: Infrastructure next steps

    - Add Nagios checks (DONE!)
    - Continue to fix bugs and other issues as they come up
    - Rebuild old Icehouse cluster as Mitaka (no upgrade)
    - Add support for object storage
    - Update documentation
    - Add support for non-live migration
    - Mellanox networking

  .. revealjs:: Project experience

    - Improve and streamline on boarding process
    - Expand cluster's disk storage capacity
    - Improve stability of the cluster
    - Add more projects!
    - Submit your request:

      - http://osuosl.org/services/powerdev/request_hosting

.. revealjs:: Questions?

  Lance Albertson

  lance@osuosl.org

  `@ramereth`_

  http://osuosl.org -- http://lancealbertson.com

  Links:

  - http://github.com/ramereth/presentation-openstack-power8
  - https://github.com/osuosl-cookbooks/osl-openstack
  - http://osuosl.org/services/powerdev/request_hosting
  - http://ftp.osuosl.org/pub/osl/repos/yum/openpower/centos-7/ppc64le/
  - http://ftp.osuosl.org/pub/osl/openpower/

  *Attribution-ShareAlike CC BY-SA ©2016*

  .. raw:: HTML

    <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
    <img alt="Creative Commons License" style="border-width:0"
    src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>

.. _@ramereth: http://twitter.com/ramereth
