Cloud Info Provider
===================

clone cip:

git clone https://recas-paas:YvCNDdic__zEnFuCjeEa@baltig.infn.it/recas-bari/paas/cip.git /opt/cip


#. Add Cloud site details


#. Copy the docker file from utils on cip directory

   cp utils/Dockerfile .

   and build it

   root@laniakea-dev-cmdb:/opt/cip# docker build -t cip:latest .

#. we need operation account on IAM, belonging to the right group and tenant.
   Create it on iam and assign it to the rignt group.
   to use it install oidc-agent.

#. edit utils/run_cip.sh

   add cmdb password and right ip address

#. edit collect and push
   with image tag and

