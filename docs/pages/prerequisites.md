# Prerequisites
## Open-source tools and bash
Open-source tools and bash must be installed on IBM i. Check out [IBM i OSS documentation](https://ibmi-oss-docs.readthedocs.io/en/latest/yum/README.html#general-information) for more information.
Make sure bash is the default shell for SSH connections - run the following command to make it so: `CALL QSYS2.SET_PASE_SHELL_INFO('*DEFAULT', '/QOpenSys/pkgs/bin/bash')`

## Code for IBM i
ARCAD-Elias for VSCode requires [Code for IBM i](https://marketplace.visualstudio.com/items?itemName=HalcyonTechLtd.code-for-ibmi) to be installed and its [requirements](https://halcyon-tech.github.io/docs/#/./README?id=requirements) met.

## ARCAD
ARCAD must be installed on the development IBM i server.
Minimum versions are:
- &gt;= 23.01 when using project mode
- At least &gt;= 13.01, preferably &gt;= 22.00 when using Skipper

## Elias REST API Server
An Elias REST API Server must be up and running and reachable by VSCode. The server package can be downloaded from [ARCAD's Customer Portal](https://portal.arcadsoftware.com/). See the [setup](/pages/setup) section for more information on how to install and configure Elias REST API Server.

## Elias CLI <small>(Project mode only)</small>
If you plan on developing your applications using Project mode, the Elias CLI must be installed on the development IBM i server. The CLI package can be downloaded from [ARCAD's Customer Portal](https://portal.arcadsoftware.com/). See the [setup](/pages/setup) section for more information on how to install Elias CLI.