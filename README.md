# OMNeT++ docker files

Source files for building OMNeT++ related docker images

 - omnetpp-base -> ubuntu based image containing all required packages to build and use OMNeT++ on the command line (clang compiler, make etc.)
   - omnetpp -> contains a pre-built OMNeT++ distribution for quickly compiling and running a model in Cmdenv (No IDE or Qtenv support!)
     - inet -> OMNeT++ and INET in the same image (release only) to be used as a base image for INET dependent models
   - omnetpp-gui -> contains a pre-built OMNeT++ with all GUI tools (except OSG and osgEarth), that can be accessed using X from the host
     - inet-gui -> Same as inet, except based on omnetpp-gui, so it has IDE and Qtenv support
 - swarm-base -> tools and packages required to run OMNeT++ simulations in a docker swarm (distcc, compiler, OMNeT++ core)
 - ci-base -> an image containing Linux, Windows and macOS compiled versions of OMNeT++ (for continuous integration)
   - ci-inet -> an image used to run continuous integration test for INET (contains NSC, ffmpeg and other optional INET dependnecies)
 - docker-sphinx -> Sphinx docker image for building OMNeT++ and INET documentation projects

> [!NOTE]
> The content of this repository is licensed under the MIT License, but
> the created images are licensed under their own respective licenses.
