Authentication & Entitlements
=============================

Identity Providers (IdPs) manage user permissions and authorization in different ways. Some systems, such as ReCaS IAM or AWS Cognito, embed the user's group memberships directly inside the access token in a JSON structure, for example:

.. code-block:: code
   
   {
   "sub": "1234567890",
   "name": "Mario Rossi",
   "group": tester
   }

Other federated AAI providers such as the Life Science Authentication and Authorization Infrastructure (LS AAI) and the European Grid Infrastructure (EGI), expose user authorization information using a more structured and complex mechanism, the ``eduPersonEntitlement``, an attribute defined in the eduPerson schema.

EDUPERSON-ENTITLEMENT
---------------------

In Research and Education, organizations have a standardised attribute to exchange informations using known schema, eduPerson is one of them. The eduPerson schema does provide an attribute called eduPersonEntitlement in which the value of the entitlement indicates a set of rights to specific resources.

The generic structure can be represented as the following:

.. code-block:: text

   urn:<authorization>:<domain>:group:<path>:<optional_role>#qualifier

In this string you can observe:

- ``URN`:` stands for uniform resource name
- ``authority:`` the authority that 'release' the entitlement 
- ``domain:`` the domain of the authority
- ``group:`` group to which a user belong
- ``role:`` the role played in that group
- ``qualifier:`` the name of the idp | qualifier

In other terms, this is a sub-category of URI that does not point to a location but rather identifies a conceptual identity or entitlement.

The two main federation that use EdupersonEntitlements that we will cover are: EGI and LS AAI.

EGI 
---

Egi entitlements follow the below schema:

.. code-block:: text

   urn:mace:egi.eu:group:<vo_name>:role=<role>#<aai-domain>

Main Elements:

- ``urn:mace:egi.eu:`` official egi namespace

- ``vo.access.egi.eu:`` name of the group or VO

- ``role=vm_operator / role=member:`` role inside the VO

- ``#aai.egi.eu:`` authority


nello script al momento ignoriamo il ruolo, prendiamo solo il nome del gruppo.

- Da riportare il codice del modulo pam -


EGI AAI (Check-in) usa lo standard eduPerson, ma segue le linee guida AARC-G002 sulla federazione per e-Infrastructure.
Questo comporta due tipi di entitlements:

a) i “res:”, (Resource Entitlements), contengono info su servizi di backend:

- urn:mace:egi.eu:res:ggus.eu

- urn:mace:egi.eu:res:gocdb#aai.egi.eu

- urn:mace:egi.eu:res:rcauth#aai.egi.eu

Questi NON rappresentano gruppi. Sono "autorizzazioni a risorse" generate dal proxy EGI per servizi partner.

b) scartati perché NON matchano :group:, poi ci sono i group


LS AAI
------

LS AAI follow a different schema: 

.. code-block:: text
   
   urn:geant:lifescience-ri.eu:group:lifescience:<subgroup/subdomain>:<service>:(<role>)#aai.lifescience-ri.eu

In the field group you find like a path where the group is the last (search if there is a subgroup) field

elementi:

- urn:geant: namespace generico per federazioni europee (GEANT)

- lifescience-ri.eu: authority LSAAI

- group: uguale a sopra

- lifescience: community

- relying_services: è il nome che LS AAI utilizza per indicare un servizio che si appoggia all’LS AAI per autenticazione - autorizzazione.

- ls_aai_ovpn_bastion: nome del gruppo vero e proprio

- role: opzionale in teoria, non ancora trovato esempi che lo includano ma dovrebbe essere possibile trovarlo

- #aai.lifescience-ri.eu: authorit


IAM Recas
---------

group taken from the token directly by the config...

AWS
---
non ci sono eduperson nativi, si posso "fabbricare":
vedi

- lamda trigger  (per imitare)
- Amazon Cognito (molto meglio, sistema molto semplice di gruppi di appartenenza ex. cognito_groups:['1','2'])

References
----------
example edupersonEntitlement: https://help.switch.ch/aai/support/documents/attributes/edupersonentitlement/
main things: https://servicedesk.surf.nl/wiki/spaces/IAM/pages/128910063/Standardized+values+for+eduPersonEntitlement
















