# Authorization

## SQL Model and Customization

The default Freenit User is defined as this:
```py
import ormar

from freenit.auth import verify
from freenit.models.sql.base import OrmarBaseModel, OrmarUserMixin, make_optional, ormar_config
from freenit.models.role import Role

class User(OrmarBaseModel, OrmarUserMixin):
    ormar_config = ormar_config.copy()

    roles = ormar.ManyToMany(Role, unique=True)

    def check(self, password: str) -> bool:
        return verify(password, self.password)


class UserOptional(User):
    pass


make_optional(UserOptional)
```

If you need a custom User model, you can copy/paste that code and add any field
to User class you need. For example, let's say the project name is `myproject`
and custom User class is needed. In that case file `myproject/models/user.py`
should look like this:

```py
class User(OrmarBaseModel, OrmarUserMixin):
    ormar_config = ormar_config.copy()

    roles = ormar.ManyToMany(Role, unique=True)
    nickname = ormar.Text()

    def check(self, password: str) -> bool:
        return verify(password, self.password)
```

## Confguration

To use the new User class you need to change `myproject/base_config.py`. By
default it looks like this:

```py
from freenit.base_config import Auth
from freenit.base_config import BaseConfig as FreenitBaseConfig


class BaseConfig(FreenitBaseConfig):
    name = "NAME"
    version = "0.0.1"


class DevConfig(BaseConfig):
    debug = True
    auth = Auth(False)
    dburl = "sqlite:///db.sqlite"


class TestConfig(BaseConfig):
    debug = True
    auth = Auth(False)
    dburl = "sqlite:///test.sqlite"


class ProdConfig(BaseConfig):
    secret = "MORESECURESECRET"
```

To make Freenit use custom User class only one line is needed:
```py
class BaseConfig(FreenitBaseConfig):
    name = "NAME"
    version = "0.0.1"
    user = 'myproject.models.user'
```

Freenit will make it so that you always import `freenit.models.user` but it will
know which module to actually use under the hood.

To customize Role, same principle applies: create model using base models and
register it in configuration.

!!! note 
    Always import from `freenit.models.user` and `freenit.models.role`, even
    when using custom classes as Freenit knows how to serve you the right
    user/role.

Nice thing is that just swapping the User and/or Role class allows you to use
Freenit's default API for auth, user, profile and roles. Of course, you can
replace them, too, but if you only need extra few fields on your class, API
overwrite is not needed.


## OpenLDAP

First thing to do is to depend on `freenit[ldap,sql]`, so change the `pyproject.toml`.

To use OpenLDAP based authentication, you need to tell Freenit where to find user and role, and
how to connect to OpenLDAP server. To do that for the whole project, you can use the following
snippet in `base_config.py`.

```py
from freenit.base_config import Auth, Mail, BaseConfig as FreenitBaseConfig
from freenit.base_config import LDAP

class BaseConfig(FreenitBaseConfig):
    name = "NAME"
    version = "0.0.1"
    user = "freenit.models.ldap.user"
    role = "freenit.models.ldap.role"

```

Then configure access to LDAP in `local_config.py`.

```py
from .base_config import DevConfig as DevConfigBase
from freenit.base_config import LDAP

class DevConfig(DevConfigBase):
    ldap = LDAP(
        host="ldap.example.com",
        service_pw="mypass",
    )
```

Do that for `TestConfig` and `ProdConfig` with different credentials. The `LDAP` class in Freenit
has following arguments with their default arguments:


```py
host="ldap.example.com"
tls=True
service_dn="cn=freenit,dc=service,dc=ldap"
service_pw=""
roleBase="dc=group,dc=ldap"
roleDN="cn={},{roleBase}"
roleClasses=["groupOfUniqueNames"]
roleMemberAttr="uniqueMember"
groupDN="cn={},{domainDN},{roleBase}"
groupClasses=["posixGroup"]
userBase="dc=account,dc=ldap"
userDN="uid={},{domainDN},{userBase}"
userClasses=["pilotPerson", "posixAccount"]
userMemberAttr="memberOf"
uidNextClass="uidNext"
uidNextDN="cn=uidnext,dc=ldap"
uidNextField="uidNumber"
gidNextClass="gidNext"
gidNextDN="cn=gidnext,dc=ldap"
gidNextField="gidNumber"
domainDN="ou={}"
domainClasses=["organizationalUnit", "pmiDelegationPath"]
```

Example of scheme that Freenit was made to work with is the following

```ldap
dn: dc=ldap
objectClass: domain

dn: cn=uidnext,dc=ldap
objectClass: uidNext
uidNumber: 10001

dn: cn=gidnext,dc=ldap
objectClass: gidNext
gidNumber: 10001

dn: dc=account,dc=ldap
objectClass: domain

dn: ou=example.com,dc=account,dc=ldap
objectClass: organizationalUnit
objectClass: pmiDelegationPath
delegationPath: /etc/certs/example.com/privkey.pem
delegationPath: /etc/certs/example.com/fullchain.pem

dn: uid=admin,ou=example.com,dc=account,dc=ldap
objectClass: pilotPerson
objectClass: posixAccount
cn: Admin
sn: User
uidNumber: 10000
gidNumber: 10000
homeDirectory: /var/mail/domains/example.com/admin
mail: admin@example.com
userClass: enabled

dn: dc=group,dc=ldap
objectClass: domain

dn: ou=example.com,dc=group,dc=ldap
objectClass: organizationalUnit

dn: cn=admin,ou=example.com,dc=group,dc=ldap
objectClass: posixGroup
cn: admin
gidNumber: 10000
memberUid: 10000

dn: dc=service,dc=ldap
objectClass: domain

dn: cn=freenit,dc=service,dc=ldap
objectClass: person
cn: Freenit
sn: Service
description: Freenit

dn: cn=admin,dc=group,dc=ldap
objectClass: groupOfUniqueNames
cn: admin
uniqueMember: uid=admin,ou=example.com,dc=account,dc=ldap
uniqueMember: cn=freenit,dc=service,dc=ldap
```
