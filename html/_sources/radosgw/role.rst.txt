======
 角色
======

A role is similar to a user and has permission policies attached to it, that determine what a role can or can not do. A role can be assumed by any identity that needs it. If a user assumes a role, a set of dynamically created temporary credentials are returned to the user. A role can be used to delegate access to users, applications, services that do not have permissions to access some s3 resources.

The following radosgw-admin commands can be used to create/ delete/ update a role and permissions asscociated with a role.

新建一个角色
------------
.. Create a Role

要新建角色，执行命令： ::

	radosgw-admin role create --role-name={role-name} [--path=="{path to the role}"] [--assume-role-policy-doc={trust-policy-document}]

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

``path``

:描述: 角色的路径，默认值是一个斜杠（ / ）。
:类型: String

``assume-role-policy-doc``

:描述: The trust relationship policy document that grants an entity permission to assume the role.
:类型: String

例如::

    radosgw-admin role create --role-name=S3Access1 --path=/application_abc/component_xyz/ --assume-role-policy-doc=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Principal\":\{\"AWS\":\[\"arn:aws:iam:::user/TESTER\"\]\},\"Action\":\[\"sts:AssumeRole\"\]\}\]\}

.. code-block:: javascript

  {
    "id": "ca43045c-082c-491a-8af1-2eebca13deec",
    "name": "S3Access1",
    "path": "/application_abc/component_xyz/",
    "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
    "create_date": "2018-10-17T10:18:29.116Z",
    "max_session_duration": 3600,
    "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
  }


删除一角色
----------
.. Delete a Role

要删除角色，执行命令： ::

	radosgw-admin role delete --role-name={role-name}

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

例如::

    radosgw-admin role delete --role-name=S3Access1

注意：在没有捆绑任何权限策略的时候才能删除此角色。


查看一角色
----------
.. Get a Role

要查看一个角色的信息，执行命令： ::

	radosgw-admin role get --role-name={role-name}

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

例如::
	
    radosgw-admin role get --role-name=S3Access1

.. code-block:: javascript

  {
    "id": "ca43045c-082c-491a-8af1-2eebca13deec",
    "name": "S3Access1",
    "path": "/application_abc/component_xyz/",
    "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
    "create_date": "2018-10-17T10:18:29.116Z",
    "max_session_duration": 3600,
    "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
  }


罗列角色
--------
.. List Roles

要罗列指定路径前缀的角色，执行命令： ::

	radosgw-admin role list [--path-prefix ={path prefix}]

请求参数
~~~~~~~~
``path-prefix``

:描述: 用于过滤角色的路径前缀。如果没指定，就罗列所有角色。
:类型: String

例如::

    radosgw-admin role list --path-prefix="/application"

.. code-block:: javascript

  [
    {
        "id": "3e1c0ff7-8f2b-456c-8fdf-20f428ba6a7f",
        "name": "S3Access1",
        "path": "/application_abc/component_xyz/",
        "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
        "create_date": "2018-10-17T10:32:01.881Z",
        "max_session_duration": 3600,
        "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
    }
  ]

更新一个角色的 assume role 策略文本
-----------------------------------
.. Update Assume Role Policy Document of a role

要修改一个角色的 assume role 策略文本，执行命令： ::

	radosgw-admin role modify --role-name={role-name} --assume-role-policy-doc={trust-policy-document}

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

``assume-role-policy-doc``

:描述: The trust relationship policy document that grants an entity permission to assume the role.
:类型: String

例如::

  radosgw-admin role modify --role-name=S3Access1 --assume-role-policy-doc=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Principal\":\{\"AWS\":\[\"arn:aws:iam:::user/TESTER2\"\]\},\"Action\":\[\"sts:AssumeRole\"\]\}\]\}

.. code-block:: javascript

  {
    "id": "ca43045c-082c-491a-8af1-2eebca13deec",
    "name": "S3Access1",
    "path": "/application_abc/component_xyz/",
    "arn": "arn:aws:iam:::role/application_abc/component_xyz/S3Access1",
    "create_date": "2018-10-17T10:18:29.116Z",
    "max_session_duration": 3600,
    "assume_role_policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"arn:aws:iam:::user/TESTER2\"]},\"Action\":[\"sts:AssumeRole\"]}]}"
  }

In the above example, we are modifying the Principal from TESTER to TESTER2 in its assume role policy document.


新增、更新一个角色的策略
------------------------
.. Add/ Update a Policy attached to a Role

To add or update the inline policy attached to a role, execute the following::

	radosgw-admin role policy put --role-name={role-name} --policy-name={policy-name} --policy-doc={permission-policy-doc}

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

``policy-name``

:描述: 策略的名字。
:类型: String

``policy-doc``

:描述: 权限策略文本。
:类型: String

例如::

  radosgw-admin role-policy put --role-name=S3Access1 --policy-name=Policy1 --policy-doc=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Action\":\[\"s3:*\"\],\"Resource\":\"arn:aws:s3:::example_bucket\"\}\]\}

In the above example, we are attaching a policy 'Policy1' to role 'S3Access1', which allows all s3 actions on 'example_bucket'.


罗列与角色捆绑的权限策略名
--------------------------
.. List Permission Policy Names attached to a Role

To list the names of permission policies attached to a role, execute the following::

	radosgw-admin role policy get --role-name={role-name}

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

例如::

  radosgw-admin role-policy list --role-name=S3Access1

.. code-block:: javascript

  [
    "Policy1"
  ]


查看与角色捆绑的权限策略
------------------------
.. Get Permission Policy attached to a Role

要查看捆绑到一个角色的具体权限策略，执行命令： ::

	radosgw-admin role policy get --role-name={role-name} --policy-name={policy-name}

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

``policy-name``

:描述: 策略的名字。
:类型: String

例如::

  radosgw-admin role-policy get --role-name=S3Access1 --policy-name=Policy1

.. code-block:: javascript

  {
    "Permission policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"s3:*\"],\"Resource\":\"arn:aws:s3:::example_bucket\"}]}"
  }


删除与角色捆绑的策略
--------------------
.. Delete Policy attached to a Role

要删除捆绑到一个角色的权限策略，执行命令： ::

	radosgw-admin role policy delete --role-name={role-name} --policy-name={policy-name}

请求参数
~~~~~~~~
``role-name``

:描述: 角色的名字。
:类型: String

``policy-name``

:描述: 策略的名字。
:类型: String

例如::

  radosgw-admin role-policy delete --role-name=S3Access1 --policy-name=Policy1


操作角色的 REST API
===================
.. REST APIs for Manipulating a Role

In addition to the above radosgw-admin commands, the following REST APIs can be used for manipulating a role. For the request parameters and their explanations, refer to the sections above.

In order to invoke the REST admin APIs, a user with admin caps needs to be created.

.. code-block:: javascript

  radosgw-admin --uid TESTER --display-name "TestUser" --access_key TESTER --secret test123 user create
  radosgw-admin caps add --uid="TESTER" --caps="roles=*"

新建角色
--------
.. Create a Role

实例::

  POST "<hostname>?Action=CreateRole&RoleName=S3Access&Path=/application_abc/component_xyz/&AssumeRolePolicyDocument=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Principal\":\{\"AWS\":\[\"arn:aws:iam:::user/TESTER\"\]\},\"Action\":\[\"sts:AssumeRole\"\]\}\]\}"

.. code-block:: XML

  <role>
    <id>8f41f4e0-7094-4dc0-ac20-074a881ccbc5</id>
    <name>S3Access</name>
    <path>/application_abc/component_xyz/</path>
    <arn>arn:aws:iam:::role/application_abc/component_xyz/S3Access</arn>
    <create_date>2018-10-23T07:43:42.811Z</create_date>
    <max_session_duration>3600</max_session_duration>
    <assume_role_policy_document>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["arn:aws:iam:::user/TESTER"]},"Action":["sts:AssumeRole"]}]}</assume_role_policy_document>
  </role>

删除角色
--------
.. Delete a Role

实例::

  POST "<hostname>?Action=DeleteRole&RoleName=S3Access"

Note: A role can be deleted only when it doesn't have any permission policy attached to it.

查看角色
--------
.. Get a Role

实例::

  POST "<hostname>?Action=GetRole&RoleName=S3Access"

.. code-block:: XML

  <role>
    <id>8f41f4e0-7094-4dc0-ac20-074a881ccbc5</id>
    <name>S3Access</name>
    <path>/application_abc/component_xyz/</path>
    <arn>arn:aws:iam:::role/application_abc/component_xyz/S3Access</arn>
    <create_date>2018-10-23T07:43:42.811Z</create_date>
    <max_session_duration>3600</max_session_duration>
    <assume_role_policy_document>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["arn:aws:iam:::user/TESTER"]},"Action":["sts:AssumeRole"]}]}</assume_role_policy_document>
  </role>

罗列角色
--------
.. List Roles

实例::

  POST "<hostname>?Action=ListRoles&RoleName=S3Access&PathPrefix=/application"

.. code-block:: XML

  <role>
    <id>8f41f4e0-7094-4dc0-ac20-074a881ccbc5</id>
    <name>S3Access</name>
    <path>/application_abc/component_xyz/</path>
    <arn>arn:aws:iam:::role/application_abc/component_xyz/S3Access</arn>
    <create_date>2018-10-23T07:43:42.811Z</create_date>
    <max_session_duration>3600</max_session_duration>
    <assume_role_policy_document>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["arn:aws:iam:::user/TESTER"]},"Action":["sts:AssumeRole"]}]}</assume_role_policy_document>
  </role>

更新 assume role 的策略文本
---------------------------
.. Update Assume Role Policy Document

实例::

  POST "<hostname>?Action=UpdateAssumeRolePolicy&RoleName=S3Access&PolicyDocument=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Principal\":\{\"AWS\":\[\"arn:aws:iam:::user/TESTER2\"\]\},\"Action\":\[\"sts:AssumeRole\"\]\}\]\}"

新增、更新与角色捆绑的策略
--------------------------
.. Add/ Update a Policy attached to a Role

实例::

    POST "<hostname>?Action=PutRolePolicy&RoleName=S3Access&PolicyName=Policy1&PolicyDocument=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Allow\",\"Action\":\[\"s3:CreateBucket\"\],\"Resource\":\"arn:aws:s3:::example_bucket\"\}\]\}"

罗列与角色捆绑的权限策略名
--------------------------
.. List Permission Policy Names attached to a Role

实例::

    POST "<hostname>?Action=ListRolePolicies&RoleName=S3Access"

.. code-block:: XML

  <PolicyNames>
    <member>Policy1</member>
  </PolicyNames>

查看与角色捆绑的权限策略
------------------------
.. Get Permission Policy attached to a Role

实例::

    POST "<hostname>?Action=GetRolePolicy&RoleName=S3Access&PolicyName=Policy1"

.. code-block:: XML

  <GetRolePolicyResult>
    <PolicyName>Policy1</PolicyName>
    <RoleName>S3Access</RoleName>
    <Permission_policy>{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["s3:CreateBucket"],"Resource":"arn:aws:s3:::example_bucket"}]}</Permission_policy>
  </GetRolePolicyResult>

删除与角色捆绑的策略
--------------------
.. Delete Policy attached to a Role

实例::

  POST "<hostname>?Action=DeleteRolePolicy&RoleName=S3Access&PolicyName=Policy1"

给角色打标
----------
.. Tag a role

A role can have multivalued tags attached to it. These tags can be passed in as part of CreateRole REST API also.
AWS 不支持有多个值的角色标签。

实例::

  POST "<hostname>?Action=TagRole&RoleName=S3Access&Tags.member.1.Key=Department&Tags.member.1.Value=Engineering"

.. code-block:: XML

  <TagRoleResponse>
    <ResponseMetadata>
      <RequestId>tx000000000000000000004-00611f337e-1027-default</RequestId>
    </ResponseMetadata>
  </TagRoleResponse>

罗列角色的标签
--------------
.. List role tags

罗列一个角色捆绑的标签。

实例::

  POST "<hostname>?Action=ListRoleTags&RoleName=S3Access"

.. code-block:: XML

  <ListRoleTagsResponse>
    <ListRoleTagsResult>
      <Tags>
        <member>
          <Key>Department</Key>
          <Value>Engineering</Value>
        </member>
      </Tags>
    </ListRoleTagsResult>
    <ResponseMetadata>
      <RequestId>tx000000000000000000005-00611f337e-1027-default</RequestId>
    </ResponseMetadata>
  </ListRoleTagsResponse>

删除角色的标签
--------------
.. Delete role tags

删除一个角色捆绑的一个或多个标签。

实例::

    POST "<hostname>?Action=UntagRoles&RoleName=S3Access&TagKeys.member.1=Department"

.. code-block:: XML

  <UntagRoleResponse>
    <ResponseMetadata>
      <RequestId>tx000000000000000000007-00611f337e-1027-default</RequestId>
    </ResponseMetadata>
  </UntagRoleResponse>

给角色打标、罗列标签和删除标签的代码示例
----------------------------------------
.. Sample code for tagging, listing tags and untagging a role

下面是使用 boto3 给一个角色打标、罗列标签、删除标签的代码示例。

.. code-block:: python

    import boto3

    access_key = 'TESTER'
    secret_key = 'test123'

    iam_client = boto3.client('iam',
        aws_access_key_id=access_key,
        aws_secret_access_key=secret_key,
        endpoint_url='http://s3.us-east.localhost:8000',
        region_name=''
    )

    policy_document = "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Federated\":[\"arn:aws:iam:::oidc-provider/localhost:8080/auth/realms/quickstart\"]},\"Action\":[\"sts:AssumeRoleWithWebIdentity\"],\"Condition\":{\"StringEquals\":{\"localhost:8080/auth/realms/quickstart:sub\":\"user1\"}}}]}"

    print ("\n Creating Role with tags\n")
    tags_list = [
        {'Key':'Department','Value':'Engineering'}
    ]
    role_response = iam_client.create_role(
        AssumeRolePolicyDocument=policy_document,
        Path='/',
        RoleName='S3Access',
        Tags=tags_list,
    )

    print ("Adding tags to role\n")
    response = iam_client.tag_role(
                RoleName='S3Access',
                Tags= [
                        {'Key':'CostCenter','Value':'123456'}
                    ]
                )
    print ("Listing role tags\n")
    response = iam_client.list_role_tags(
                RoleName='S3Access'
                )
    print (response)
    print ("Untagging role\n")
    response = iam_client.untag_role(
        RoleName='S3Access',
        TagKeys=[
            'Department',
        ]
    )
