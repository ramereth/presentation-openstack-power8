Building and running OpenStack on POWER8
========================================

HTML presentation on building and running OpenStack on POWER8.

To build and view, do the following:

::

  virtualenv venv
  pip install -r requirements.txt
  make html

Proposal
--------

Over the past two years the OSL has been building a POWER8 based OpenStack
environment working in conjunction with IBM. The purpose of this environment is
to provide a stable yet flexible infrastructure for FOSS projects to port and
test their code on the new PowerPC 64bit Little Endian (ppc64le) architecture. 

This session will cover various aspects of the path we took to build and
continue to run the environment. Some topics will include some of the initial
challenges we faced and how we solved them. In addition, we’ll cover some of the
specific porting issues we ran into with ppc64le. We’ll also cover some of the
major issues we ran into with OpenStack specifically on ppc64le and how we
solved them. And finally, we’ll discuss the future of the cluster and the work
we’ve put into designing it.

Attendees to this session should have some foundational knowledge of the POWER
and OpenStack ecosystem. If you’re interested in architecture porting issues
and/or interested in OpenStack deployment war stories, this session might be for
you.

Summary
-------
* Building a platform based on Fedora and Openstack
* Key porting issues we ran into
* Problems we have encountered with OpenStack
* Current state of the cluster
* CentOS 7 ppc64le on OpenStack
* Next generation of the cluster
