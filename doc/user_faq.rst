..
  Copyright (C) 2014  Red Hat, Inc.

  This copyrighted material is made available to anyone wishing to use,
  modify, copy, or redistribute it subject to the terms and conditions of
  the GNU General Public License v.2, or (at your option) any later version.
  This program is distributed in the hope that it will be useful, but WITHOUT
  ANY WARRANTY expressed or implied, including the implied warranties of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General
  Public License for more details.  You should have received a copy of the
  GNU General Public License along with this program; if not, write to the
  Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA
  02110-1301, USA.  Any Red Hat trademarks that are incorporated in the
  source code or documentation are not subject to the GNU General Public
  License and may only be used or replicated with the express permission of
  Red Hat, Inc.

################
 DNF User's FAQ
################

.. contents::

=================
General Questions
=================

What does DNF stand for?
========================

Dandified `Yum <http://yum.baseurl.org/>`_.

What to do with packages that DNF refuses to remove because their ``%pre`` or ``%preun`` scripts are failing?
=============================================================================================================

If this happens, it is a packaging error and consider reporting the failure to
the package's maintainer.

You can usually remove such package with ``rpm``::

    rpm -e <package-version> --noscripts

Why are ``dnf check-update`` packages not marked for upgrade in the following ``dnf upgrade``
=============================================================================================

Sometimes one can see that a newer version of a package is available in the repos::

    $ dnf check-update
    libocsync0.x86_64 0.91.4-2.1              devel_repo
    owncloud-client.x86_64 1.5.0-18.1         devel_repo

Yet the immediately following ``dnf upgrade`` does not offer them for upgrade::

    $ dnf upgrade
    Resolving dependencies
    --> Starting dependency resolution
    --> Finished dependency resolution
    Dependencies resolved.
    Nothing to do.

It might seem odd but in fact this can happen quite easily: what the first command does is only check whether there are some available packages with the same name as an installed package but with a higher version. Those are considered upgrade candidates by ``check-update``, but no actual dependency resolving takes place there. That only happens during ``dnf upgrade`` and if the resolving procedure then discovers that some of the packages do not have their dependencies ready yet, then they are not offered in the upgrade. To see the precise reason why it was not possible to do the upgrade in this case, use::

    $ dnf upgrade --best

Why do I get different results with ``dnf update`` vs ``yum update``?
=====================================================================

We get this reported as a bug quite often, but it usually is not. One reason to see this is that DNF does not list update candidates as it explores them. More frequently however the reporter means actual difference in the proposed transaction. This is most often because the metadata the two packagers are working with were taken at a different time (DNF has a notoriously looser schedule on metadata updates to save time and bandwidth), and sometimes also because the depsolvers inside are designed to take a different course of action when encountering some specific update scenario.

The bottom line is: unless a real update problem occurs (i.e. DNF refuses to update a package that Yum updates) with the same set of metadata, this is not an issue.

Is it possible to force DNF to get the latest metadata on ``dnf upgrade``?
==============================================================================

Yes, clear the cache first::

    $ dnf clean metadata
    $ dnf upgrade

An alternative is to shorten the default expiry time of repos, for that edit ``/etc/dnf/dnf.conf`` and set::

    metadata_expire=0

Of course, some repos might use a custom ``metadata_expire`` value, you'll currently have to change these manually too.

If you're the kind of the user who always wants the freshest metadata possible, you'll probably want to :ref:`disable the automatic MD updates <disabling_makecache_service-label>`.

.. _disabling_makecache_service-label:

How do I disable automatic metadata synchronization service?
============================================================

Several ways to do that. The DNF way is to add the following to ``/etc/dnf/dnf.conf``::

    metadata_timer_sync=0

Shouldn't DNF exit soon from certain commands if it is not run under root?
==========================================================================

No, there can be systems and scenarios that allow other users than root to successfully perform ``dnf intall`` and similar and it would be impractical to stop these from functioning by the UID check. Alternatively, the practice of checking filesystem permissions instead of the effective UID could lead to false positives since there is plenty of time between DNF startup and the possible transaction start when permissions can be changed by a different process.

If the time loss incurred by repeated runs of DNF is unacceptable for you, consider using the `noroot plugin <https://github.com/akozumpl/dnf-plugins-core/blob/master/plugins/noroot.py>`_.

===================
Using DNF in Fedora
===================

For my stable Fedora release, can I install the rawhide packages for testing purposes?
======================================================================================

Yes, in two steps: first install the necessary ``.repo`` files::

    dnf install fedora-release-rawhide

Then, when you want to include the packages from the rawhide repo, execute a DNF command with Rawhide enabled::

    dnf install --enablerepo=rawhide upgrade rpm

.. note::

    Installing rawhide packages onto a stable Fedora release system is generally discouraged as it leads to less tested combinations of installed packages. Please consider this step carefully.
