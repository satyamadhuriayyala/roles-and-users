# roles-and-users
zoho online programming round

TERMINOLOGY:
This section defines OPSS terms used throughout this document.

USERS AND GROUPS:

A user is an end-user accessing a service. A group is a subset of users and other groups, so a group has a hierarchical structure.
An authenticated user is a user whose credentials have been validated. To authenticate users, OPSS uses WebLogic Server Authentication providers.

An anonymous user is an authenticated user who is permitted access to only unprotected resources.

Application Stripe

An application stripe or stripe (when the application is understood) is a subset of policies. An application chooses the policies to use by specifying a stripe in the security store. Several applications can use the same stripe.

ROLES:

An enterprise role or enterprise group is a collection of users and groups.

An application role is a collection of users and other application roles. Application roles are defined in application policies and they are not necessarily known to a Java container.

Application roles can be mapped many-to-many to enterprise roles. For example, the employee enterprise role (in the identity store) can be mapped to the helpdesk service request application role (in one stripe) and to the self service HR application role (in some other stripe).

PRINCIPAL:

A principal is the identity to which a policy grants permissions. A principal can be a user, an external role, or an application role.

POLICIES:

An application policy is a policy that specifies a set of permissions for principals, such as viewing application web pages or modifying application reports. Application policies must have at least one principal and can specify either separate permissions or a permission set, but not both.

A system policy is a policy that specifies a set of permissions for a principal or a codesource. The scope of system policies is the entire domain. System policies grant permissions to codesources and principals, while application policies grant permissions to principals only. Application and system policies are specified in the jazn-data.xml file.

SUBJECT:

The subject is a collection of principals and user credentials, such as passwords and cryptographic keys. The authentication service populates the subject with users, groups, application roles, and credentials. The OPSS subject is critical in identity propagation across domains.

OPSS Configuration File

The OPSS configuration file where all security services for the entire domain are specified. The specifications in the OPSS configuration file apply to all servers and applications running in the domain.

By default, the OPSS configuration file is the jps-config.xml file (for Java EE applications) and the jps-config-jse.xml file (for Java SE applications) and they are located in the $DOMAIN_HOME/config/fmwconfig directory.

OPSS CONTEXT:

The context defines a collection of services instances common to a domain. Contexts are specified in the OPSS configuration file.

SECURITY STORES:

The identity store is the repository of enterprise users and groups and must be LDAP. Ready-to-use, users and groups can be stored in the default provider (embedded LDAP server).

The security store is the repository of system and application policies, credentials, audit data, keys, and certificates used by all applications running in a domain.

The policy store refers to the portion or the security store where application and system policies are kept. The credential store refers to the portion of the security store where credentials are kept. The keystore refers to the portion of the security store where keys and certificates are kept. The truststore refers to a keystore where trusted certificates of third-party certificate authorities are kept.

ROLE MAPPING:

OPSS supports the many-to-many mapping of application roles to enterprise groups. This allows users in enterprise groups to access application resources as specified by application roles.

Mapping an application role to an enterprise group rewrites the permission of the enterprise group as the union of its permissions and those of the mapped role. Therefore, the mapping may augment the permissions of the enterprise group but never removes any permission from it.

After deploying the application, you map application roles to enterprise groups with WebLogic Scripting Tool (WLST), Oracle Enterprise Manager Fusion Middleware Control (Fusion Middleware Control), or Oracle Entitlements Server (OES).

Permission Inheritance and the Role Hierarchy:

Roles can be structured hierarchically by the relation “is a member of," and a role can have as members users or roles.

In a role hierarchy, role members inherit permissions from the parent role. Thus, if roleA is a member of roleB, then all permissions granted to roleB are also granted to roleA. In addition, roleA may have its own particular permissions.

Role Hierarchy Example
The following example illustrates a role hierarchy of nested application users and roles. In the example, the developerAppRole role has the following members:

1.developer
2.developer_group
3.managerAppRole
4.directorAppRole

The directorAppRole role has the following members:

1.developer
2.developer_group

The relevant portions of the jazn-data.xml file specifying this hierarchy follows:

<policy-store>
  <applications>
    <application>
      <name>MyApp</name>
      <app-roles>
        <app-role>
          <name>developerAppRole</name>
          <class>oracle.security.jps.service.policystore.ApplicationRole</class>
          <display-name>Application developer role</display-name>
          <description>Application developer role</description>
          <guid>61FD29C0D47E11DABF9BA765378CF9F5</guid>
          <members>
            <member>
                      <class>weblogic.security.principal.WLSUserImpl</class>
                      <name>developer</name>
            </member>
            <member>
                      <class>weblogic.security.principal.WLSUserImpl</class>
                      <name>directorAppRole</name>
                        <name>directorAppRole</name>
            </member>
            <member>
                      <class>weblogic.security.principal.WLSGroupImpl</class>
                      <name>developer_group</name>
            </member>
            <member>
              <class>
oracle.security.jps.service.policystore.ApplicationRole</class>
              <name>managerAppRole</name>
            </member>
          </members>
        </app-role>
        <app-role>
                          <name>directorAppRole</name>
                          <class>oracle.security.jps.service.policystore.ApplicationRole</class>
                          <display-name>Application director role </display-name>
                          <description>Application director role</description>
                          <guid>61FD29C0D47E11DABF9BA765378CF9F8</guid>
                          <members>
            <member>
                                      <class>weblogic.security.principal.WLSUserImpl</class>
                                      <name>developer</name>
            </member>
            <member>
                                        <class>weblogic.security.principal.WLSGroupImpl</class>
                                        <name>developer_group</name>
            </member>
                                   </members>
         </app-role> ...
       </app-roles>

      <jazn-policy>
        <grant>
          <grantee>
             <principals>
                <principal>
                   <class>
           oracle.security.jps.service.policystore.ApplicationRole</class>
                   <name>developerAppRole</name>
                </principal>
             </principals>
          </grantee>
          <permissions>
            <permission>
              <class>java.io.FilePermission</class>
              <name>/tmp/oracle.txt</name>
              <actions>write</actions>
             </permission>
           </permissions>
         </grant>

         <grant>
           <grantee>
             <principals>
               <principal>
                 <class>
           oracle.security.jps.service.policystore.ApplicationRole</class>
                 <name>managerAppRole</name>
               </principal>
             </principals>
           </grantee>
           <permissions>
             <permission>
               <class>java.util.PropertyPermission</class>
               <name>myProperty</name>
               <actions>read</actions>
             </permission>
            </permissions>

           </grant>
           <grant>
             <grantee>
               <principals>
                 <principal>
                   <class>
oracle.security.jps.service.policystore.ApplicationRole</class>
                   <name>directorAppRole</name>
                 </principal>
               </principals>
             </grantee>
             <permissions>
               <permission>
                 <class>foo.CustomPermission</class>
                 <name>myProperty</name>
                 <actions>*</actions>
               </permission>
             </permissions>
           </grant>
         </jazn-policy>
       </policy-store>

    summarizes the permissions of the five users and roles in the example.
 Granted and Inherited Permissions

Role	Permission Granted	Actual Permissions:

developerAppRole      P1=java.io.FilePermission                  P1

managerAppRole        P2= java.util.PropertyPermission           P2 and (inherited) P1

directorAppRole       P3=foo.CustomPermission                    P3 and (inherited) P1

developer             NA                                         P1 and P3 (both inherited)

developer_group       NA                                         P1 and P3 (both inherited)

About the Role Category
A role category is a collection of application roles. A role category contains no hierarchical structure.

The following example illustrates the configuration of a role category:

<app-roles>
  <app-role>
    <name>AppRole_READONLY</name>
    <display-name>display name</display-name>
    <description>description</description>
    <class>oracle.security.jps.service.policystore.ApplicationRole</class>
    <extended-attributes>
      <attribute>
        <name>ROLE_CATEGORY</name>
        <values>
          <value>RC_READONLY</value>
        </values>
      </attribute>
    </extended-attributes>
  </app-role>
</app-roles>
<role-categories>
  <role-category>
    <name>RC_READONLY</name>
    <display-name>RC_READONLY display name</display-name>
    <description>RC_READONLY description</description>
  </role-category>
</role-categories>
The members of a role category are not configured within <role-category> but within <extended-attributes> of the corresponding application role. The role category name is case-insensitive. The role category can be managed with the RoleCategoryManager interface.
    
  About the Anonymous User and Role:
OPSS supports the anonymous user and the anonymous role. The permissions granted to them follow from the enterprise groups and application roles of which they are a member, that is, they are granted inherited permissions only. A typical use of the anonymous user and role is to allow unauthenticated users to access public, unprotected resources. You can configure this role in the Java servlet filter and the EJB interceptor.
