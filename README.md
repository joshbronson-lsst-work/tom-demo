# TOM Toolkit Demonstration TOM
To see the demonstration TOM, visit http://tom-demo.lco.global.

<!--- _(If you are interested in exploring a vanilla, unadorned, out-of-the-box TOM, visit http://tom-base.lco.global)._ --->

This demonstration TOM, with its source code, goes beyond the basic, out-of-the-box TOM and  is designed to provide
examples of how to use the features provided by the [TOM Toolkit](https://tom-toolkit.readthedocs.io/) and its 
associated apps.

The current version of TOM-Demo includes the following apps:
* [tom_base](https://github.com/TOMToolkit/tom_base)
* [tom_swift](https://github.com/TOMToolkit/tom_swift)
* [tom_hermes](https://github.com/TOMToolkit/tom_hermes)
* [tom_fink](https://github.com/TOMToolkit/tom_fink)
* [tom_tns](https://github.com/TOMToolkit/tom_tns)
* [tom_registration](https://github.com/TOMToolkit/tom_registration)

Register with a username with the demonstration TOM to see the full suite of features.

(Note: Observation submission has been deactivated, see [the docs](https://tom-toolkit.readthedocs.io/en/stable/api/tom_observations/facilities.html) for informaiton on how to configure facilities in your own TOM.)

# Building your own TOM
If you are interested in implementing your own TOM using the TOM Toolkit, you may also be interested in the following:
* [TOM Toolkit Documentation](https://tom-toolkit.readthedocs.io/)
* [Joining the TOM Toolkit Slack](https://join.slack.com/t/tom-toolkit/shared_invite/zt-28ameesvb-QK5o~zWLlnL_Zmh22izKgg)
* [Contact the TOM Toolkit Developers](tomtoolkit-maintainers@lco.global)

# Creating an Externally Facing TOM
Creating an externally-facing TOM can be done in a few different
ways. The basic ingredents include the following:
* a domain name and control over DNS entries for that domain
* a platform on which you can spin up a Kubernetes cluster and a
  static IP address.

There are many ways to gather these ingredients, and users should
decide for themselves how to gather them. However, there are
instructions [here](deployment.md) to deploy an external TOM if you
are willing to use the following kinds of ingredients:
* a domain name and control over DNS entries for that domain:
  Squarespace. (You can easily use your own DNS server, though, if you
  have the ability to create DNS A records for hosts for which you
  have matching TLS keys at your organization.)
* a platform on which you can spin up a Kubernetes cluster and a
  static IP address: Google Compute Platform

This isn't an endorsement of these particular tools, and many other
fine alternatives exist for each of them.
