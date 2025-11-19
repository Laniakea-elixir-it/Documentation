EDUPERSON-ENTITLEMENT
=====================
In Research and Education organizations have standardised attribute to exchange informations using know schema, eduPerson is one of them. The eduPerson schema does provide an attribute called eduPersonEntitlement in which the value of the entitlement indicates a set of rights to specific resources.

The generic structure can be represented as the following:

.. code-block:: text

   urn:<authorization>:<domain>:group:<path>:<optional_role>#qualifier

In this string you can observe:

- URN, that stands for uniform resource name
- the authority that 'release' the entitlement 
- the domain of the authority
- the group to which a user belong
- the role in that group
- the qualifier / idp

In other terms, this is a sub-category of URI that does not point to a location but rather identifies a conceptual identity or entitlement.

EGI
---
gli entitlement egi seguono questo schema:

.. code-block:: text

   urn:mace:egi.eu:group:<vo_name>:role=<role>#<aai-domain>


Elementi importanti:

- urn:mace:egi.eu: namespace ufficiale di egi

- group: indica il gruppo della VO

- vo.access.egi.eu: nome della VO o del gruppo

- role=vm_operator / role=member: ruolo all'interno della VO

- #aai.egi.eu: authority


nello script al momento ignoriamo il ruolo, prendiamo solo il nome del gruppo 


EGI AAI (Check-in) usa lo standard eduPerson, ma segue le linee guida AARC-G002 sulla federazione per e-Infrastructure.
Questo comporta due tipi di entitlements:

a) i “res:”, (Resource Entitlements), contengono info su servizi di backend:

- urn:mace:egi.eu:res:ggus.eu

- urn:mace:egi.eu:res:gocdb#aai.egi.eu

- urn:mace:egi.eu:res:rcauth#aai.egi.eu

Questi NON rappresentano gruppi. Sono "autorizzazioni a risorse" generate dal proxy EGI per servizi partner.

b) scartati perché NON matchano :group:, poi ci sono i group


LSAAI
-----

schema: 

.. code-block:: text
   
   urn:geant:lifescience-ri.eu:group:lifescience:<subgroup/subdomain>:<service>:(<role>)#aai.lifescience-ri.eu

elementi:

- urn:geant: namespace generico per federazioni europee (GEANT)

- lifescience-ri.eu: authority LSAAI

- group: uguale a sopra

- lifescience: community

- relying_services: è il nome che LS AAI utilizza per indicare un servizio che si appoggia all’LS AAI per autenticazione - autorizzazione.

- ls_aai_ovpn_bastion: nome del gruppo vero e proprio

- role: opzionale in teoria, non ancora trovato esempi che lo includano ma dovrebbe essere possibile trovarlo

- #aai.lifescience-ri.eu: authorit

TODO
----
per far saltare fuori i gruppi usando egi basta scartare i ruoli, e con la regex ``.*:group:([^#]+)`` si ottiene facilmente solo il gruppo (vo.access.egi.eu
 ad esempio). Per lsaai invece si otterebbe la stringa/path, (lifescience:relying_services:ls_aai_ovpn_bastion) di cui ci interessa solo ultimo valore/o penultimo. (DA CONTROLLARE se è possbile avere anche il role, perchè se si è un problema, dato che non si può più selezionare semplicemente l'ultimo campo dopo gli ultimi :) -> (flag, 1 + 3*TRUE/FALSE ??)

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
















