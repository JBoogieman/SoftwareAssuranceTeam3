# TMT ID crosswalk

Our Part 1 and Part 2 tables in [Designing for Software Security Engineering](../Designing-for-Software-Security-Engineering.md) use the Threat Modeling Tool's own threat IDs, the ID column in TMT's Analysis View. The [HTML report](keycloak-threat-model-report.htm) numbers the same 193 threats 1–193 in its own order, grouped by interaction. Use this table to find any of our IDs in the report.

| TMT ID | Report # | Interaction | Flow (report heading) | Threat | Category | State |
|---|---|---|---|---|---|---|
| 1 | 32 | 1, 3 | Authorization request | Cross Site Scripting | Tampering | Mitigation Implemented |
| 2 | 33 | 1, 3 | Authorization request | Elevation Using Impersonation | Elevation Of Privilege | Mitigation Implemented |
| 3 | 143 | 3 | Token request | Spoofing the Client application External Entity | Spoofing | Mitigation Implemented |
| 4 | 144 | 3 | Token request | Cross Site Scripting | Tampering | Not Applicable |
| 5 | 145 | 3 | Token request | Elevation Using Impersonation | Elevation Of Privilege | Needs Investigation |
| 21 | 57 | 4 | Broker login request | Spoofing the End user (browser) External Entity | Spoofing | Mitigation Implemented |
| 22 | 58 | 4 | Broker login request | Cross Site Scripting | Tampering | Mitigation Implemented |
| 23 | 59 | 4 | Broker login request | Elevation Using Impersonation | Elevation Of Privilege | Mitigation Implemented |
| 24 | 128 | 4 | SAML authentication request | Weakness in SSO Authorization | Elevation Of Privilege | Mitigation Implemented |
| 25 | 132 | 4 | SAML response | Spoofing the Partner identity provider External Entity | Spoofing | Mitigation Implemented |
| 26 | 133 | 4 | SAML response | Cross Site Scripting | Tampering | Mitigation Implemented |
| 27 | 134 | 4 | SAML response | Elevation Using Impersonation | Elevation Of Privilege | Needs Investigation |
| 35 | 66 | 4 | Brokered identity context | Broker external login Process Memory Tampered | Tampering | Not Applicable |
| 36 | 67 | 4 | Brokered identity context | Cross Site Scripting | Tampering | Mitigation Implemented |
| 37 | 68 | 4 | Brokered identity context | Elevation Using Impersonation | Elevation Of Privilege | Mitigation Implemented |
| 45 | 77 | 3 | Client session update | Spoofing of Destination Data Store Session store | Spoofing | Mitigation Implemented |
| 46 | 76 | 3 | Client session update | Potential Excessive Resource Consumption for Issue tokens or Session store | Denial Of Service | Mitigation Implemented |
| 47 | 30 | 1, 3 | Authorization code record | Spoofing of Destination Data Store Session store | Spoofing | Mitigation Implemented |
| 48 | 29 | 1, 3 | Authorization code record | Potential Excessive Resource Consumption for Authenticate user or Session store | Denial Of Service | Mitigation Implemented |
| 49 | 81 | 3 | Code record and client session | Spoofing of Source Data Store Session store | Spoofing | Mitigation Implemented |
| 50 | 80 | 3 | Code record and client session | Cross Site Scripting | Tampering | Not Applicable |
| 51 | 79 | 3 | Code record and client session | Persistent Cross Site Scripting | Tampering | Not Applicable |
| 52 | 78 | 3 | Code record and client session | Weak Access Control for a Resource | Information Disclosure | Mitigation Implemented |
| 53 | 75 | 3 | Client registration and signing keys | Spoofing of Source Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 54 | 74 | 3 | Client registration and signing keys | Cross Site Scripting | Tampering | Not Applicable |
| 55 | 73 | 3 | Client registration and signing keys | Persistent Cross Site Scripting | Tampering | Not Applicable |
| 56 | 72 | 3 | Client registration and signing keys | Weak Access Control for a Resource | Information Disclosure | Needs Investigation |
| 57 | 96 | 4 | Identity provider configuration and mappers | Spoofing of Source Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 58 | 99 | 4 | Identity provider configuration and mappers | Cross Site Scripting | Tampering | Mitigation Implemented |
| 59 | 98 | 4 | Identity provider configuration and mappers | Persistent Cross Site Scripting | Tampering | Mitigation Implemented |
| 60 | 97 | 4 | Identity provider configuration and mappers | Weak Access Control for a Resource | Information Disclosure | Mitigation Implemented |
| 61 | 142 | 3 | Token events | Spoofing of Destination Data Store Event store | Spoofing | Mitigation Implemented |
| 62 | 141 | 3 | Token events | Potential SQL Injection Vulnerability for Event store | Tampering | Mitigation Implemented |
| 63 | 140 | 3 | Token events | Potential Excessive Resource Consumption for Issue tokens or Event store | Denial Of Service | Mitigation Implemented |
| 64 | 71 | 4 | Brokered user, identity link, and mapped roles | Spoofing of Destination Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 65 | 70 | 4 | Brokered user, identity link, and mapped roles | Potential SQL Injection Vulnerability for Keycloak database | Tampering | Mitigation Implemented |
| 66 | 69 | 4 | Brokered user, identity link, and mapped roles | Potential Excessive Resource Consumption for Broker external login or Keycloak database | Denial Of Service | Mitigation Implemented |
| 67 | 160 | 1 | User credentials and client registration | Spoofing of Source Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 68 | 159 | 1 | User credentials and client registration | Cross Site Scripting | Tampering | Mitigation Implemented |
| 69 | 158 | 1 | User credentials and client registration | Persistent Cross Site Scripting | Tampering | Mitigation Implemented |
| 70 | 157 | 1 | User credentials and client registration | Weak Access Control for a Resource | Information Disclosure | Mitigation Implemented |
| 71 | 56 | 4 | Broker login events | Spoofing of Destination Data Store Event store | Spoofing | Mitigation Implemented |
| 72 | 55 | 4 | Broker login events | Potential SQL Injection Vulnerability for Event store | Tampering | Mitigation Implemented |
| 73 | 54 | 4 | Broker login events | Potential Excessive Resource Consumption for Broker external login or Event store | Denial Of Service | Mitigation Implemented |
| 83 | 39 | 1, 3 | Authorization request | Potential Data Repudiation by Authenticate user | Repudiation | Mitigation Implemented |
| 84 | 38 | 1, 3 | Authorization request | Potential Process Crash or Stop for Authenticate user | Denial Of Service | Mitigation Implemented |
| 85 | 37 | 1, 3 | Authorization request | Data Flow Authorization request Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 86 | 36 | 1, 3 | Authorization request | Authenticate user May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 87 | 35 | 1, 3 | Authorization request | Elevation by Changing the Execution Flow in Authenticate user | Elevation Of Privilege | Mitigation Implemented |
| 88 | 34 | 1, 3 | Authorization request | Cross Site Request Forgery | Elevation Of Privilege | Mitigation Implemented |
| 89 | 42 | 1, 3 | Authorization response | Spoofing of the End user (browser) External Destination Entity | Spoofing | Mitigation Implemented |
| 90 | 41 | 1, 3 | Authorization response | External Entity End user (browser) Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 91 | 40 | 1, 3 | Authorization response | Data Flow Authorization response Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 92 | 146 | 3 | Token request | Potential Data Repudiation by Issue tokens | Repudiation | Mitigation Implemented |
| 93 | 147 | 3 | Token request | Potential Process Crash or Stop for Issue tokens | Denial Of Service | Mitigation Implemented |
| 94 | 148 | 3 | Token request | Data Flow Token request Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 95 | 149 | 3 | Token request | Issue tokens May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 96 | 150 | 3 | Token request | Elevation by Changing the Execution Flow in Issue tokens | Elevation Of Privilege | Mitigation Implemented |
| 97 | 151 | 3 | Token response | Spoofing of the Client application External Destination Entity | Spoofing | Mitigation Implemented |
| 98 | 152 | 3 | Token response | External Entity Client application Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 99 | 153 | 3 | Token response | Data Flow Token response Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 100 | 60 | 4 | Broker login request | Potential Data Repudiation by Broker external login | Repudiation | Mitigation Implemented |
| 101 | 61 | 4 | Broker login request | Potential Process Crash or Stop for Broker external login | Denial Of Service | Mitigation Implemented |
| 102 | 62 | 4 | Broker login request | Data Flow Broker login request Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 103 | 63 | 4 | Broker login request | Broker external login May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 104 | 64 | 4 | Broker login request | Elevation by Changing the Execution Flow in Broker external login | Elevation Of Privilege | Mitigation Implemented |
| 105 | 65 | 4 | Broker login request | Cross Site Request Forgery | Elevation Of Privilege | Mitigation Implemented |
| 106 | 129 | 4 | SAML authentication request | Spoofing of the Partner identity provider External Destination Entity | Spoofing | Mitigation Implemented |
| 107 | 130 | 4 | SAML authentication request | External Entity Partner identity provider Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 108 | 131 | 4 | SAML authentication request | Data Flow SAML authentication request Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 109 | 135 | 4 | SAML response | Potential Data Repudiation by Broker external login | Repudiation | Needs Investigation |
| 110 | 136 | 4 | SAML response | Potential Process Crash or Stop for Broker external login | Denial Of Service | Mitigation Implemented |
| 111 | 137 | 4 | SAML response | Data Flow SAML response Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 112 | 138 | 4 | SAML response | Broker external login May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 113 | 139 | 4 | SAML response | Elevation by Changing the Execution Flow in Broker external login | Elevation Of Privilege | Mitigation Implemented |
| 172 | 31 | 1, 3 | Authorization request | Spoofing the End user (browser) External Entity | Spoofing | Mitigation Implemented |
| 182 | 107 | 1 | OTP challenge | Spoofing of the End user (browser) External Destination Entity | Spoofing | Needs Investigation |
| 183 | 108 | 1 | OTP challenge | External Entity End user (browser) Potentially Denies Receiving Data | Repudiation | Needs Investigation |
| 184 | 109 | 1 | OTP challenge | Data Flow OTP challenge Is Potentially Interrupted | Denial Of Service | Needs Investigation |
| 185 | 118 | 1 | OTP response (code) | Spoofing the End user (browser) External Entity | Spoofing | Needs Investigation |
| 186 | 117 | 1 | OTP response (code) | Cross Site Scripting | Tampering | Needs Investigation |
| 187 | 116 | 1 | OTP response (code) | Potential Data Repudiation by Authenticate user | Repudiation | Needs Investigation |
| 188 | 115 | 1 | OTP response (code) | Potential Process Crash or Stop for Authenticate user | Denial Of Service | Needs Investigation |
| 189 | 114 | 1 | OTP response (code) | Data Flow OTP response (code) Is Potentially Interrupted | Denial Of Service | Needs Investigation |
| 190 | 113 | 1 | OTP response (code) | Elevation Using Impersonation | Elevation Of Privilege | Needs Investigation |
| 191 | 112 | 1 | OTP response (code) | Authenticate user May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Needs Investigation |
| 192 | 111 | 1 | OTP response (code) | Elevation by Changing the Execution Flow in Authenticate user | Elevation Of Privilege | Needs Investigation |
| 193 | 110 | 1 | OTP response (code) | Cross Site Request Forgery | Elevation Of Privilege | Needs Investigation |
| 197 | 26 | 1 | Authentication events | Spoofing of Destination Data Store Event store | Spoofing | Needs Investigation |
| 198 | 27 | 1 | Authentication events | Potential SQL Injection Vulnerability for Event store | Tampering | Needs Investigation |
| 199 | 28 | 1 | Authentication events | Potential Excessive Resource Consumption for Authenticate user or Event store | Denial Of Service | Needs Investigation |
| 209 | 187 | 1 | Username and password submission | Spoofing the End user (browser) External Entity | Spoofing | Needs Investigation |
| 210 | 186 | 1 | Username and password submission | Cross Site Scripting | Tampering | Needs Investigation |
| 211 | 185 | 1 | Username and password submission | Potential Data Repudiation by Authenticate user | Repudiation | Needs Investigation |
| 212 | 184 | 1 | Username and password submission | Potential Process Crash or Stop for Authenticate user | Denial Of Service | Needs Investigation |
| 213 | 183 | 1 | Username and password submission | Data Flow Username and password submission Is Potentially Interrupted | Denial Of Service | Needs Investigation |
| 214 | 182 | 1 | Username and password submission | Elevation Using Impersonation | Elevation Of Privilege | Needs Investigation |
| 215 | 181 | 1 | Username and password submission | Authenticate user May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Needs Investigation |
| 216 | 180 | 1 | Username and password submission | Elevation by Changing the Execution Flow in Authenticate user | Elevation Of Privilege | Needs Investigation |
| 217 | 179 | 1 | Username and password submission | Cross Site Request Forgery | Elevation Of Privilege | Needs Investigation |
| 218 | 82 | 5 | Credential validation request | Authenticate user Process Memory Tampered | Tampering | Not Applicable |
| 219 | 83 | 5 | Credential validation request | Cross Site Scripting | Tampering | Not Applicable |
| 220 | 84 | 5 | Credential validation request | Elevation Using Impersonation | Elevation Of Privilege | Mitigation Implemented |
| 221 | 103 | 5 | LDAP bind request | Spoofing of the Active Directory External Destination Entity | Spoofing | Mitigation Implemented |
| 222 | 104 | 5 | LDAP bind request | External Entity Active Directory Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 223 | 105 | 5 | LDAP bind request | Data Flow LDAP bind request Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 224 | 106 | 5 | LDAP bind request | Weakness in SSO Authorization | Elevation Of Privilege | Mitigation Implemented |
| 225 | 43 | 5 | Bind result | Spoofing the Synchronize federated users Process | Spoofing | Mitigation Implemented |
| 226 | 44 | 5 | Bind result | Spoofing the Active Directory External Entity | Spoofing | Mitigation Implemented |
| 227 | 45 | 5 | Bind result | Potential Lack of Input Validation for Synchronize federated users | Tampering | Mitigation Implemented |
| 228 | 46 | 5 | Bind result | Cross Site Scripting | Tampering | Not Applicable |
| 229 | 47 | 5 | Bind result | Potential Data Repudiation by Synchronize federated users | Repudiation | Mitigation Implemented |
| 230 | 48 | 5 | Bind result | Data Flow Sniffing | Information Disclosure | Mitigation Implemented |
| 231 | 49 | 5 | Bind result | Potential Process Crash or Stop for Synchronize federated users | Denial Of Service | Mitigation Implemented |
| 232 | 50 | 5 | Bind result | Data Flow Bind result Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 233 | 51 | 5 | Bind result | Elevation Using Impersonation | Elevation Of Privilege | Mitigation Implemented |
| 234 | 52 | 5 | Bind result | Synchronize federated users May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 235 | 53 | 5 | Bind result | Elevation by Changing the Execution Flow in Synchronize federated users | Elevation Of Privilege | Mitigation Implemented |
| 236 | 85 | 5 | Directory search request | Spoofing of the Active Directory External Destination Entity | Spoofing | Mitigation Implemented |
| 237 | 86 | 5 | Directory search request | External Entity Active Directory Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 238 | 87 | 5 | Directory search request | Data Flow Directory search request Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 239 | 88 | 5 | Directory search request | Weakness in SSO Authorization | Elevation Of Privilege | Mitigation Implemented |
| 240 | 161 | 5 | User entries and group memberships | Spoofing the Synchronize federated users Process | Spoofing | Mitigation Implemented |
| 241 | 162 | 5 | User entries and group memberships | Spoofing the Active Directory External Entity | Spoofing | Mitigation Implemented |
| 242 | 163 | 5 | User entries and group memberships | Potential Lack of Input Validation for Synchronize federated users | Tampering | Mitigation Implemented |
| 243 | 164 | 5 | User entries and group memberships | Cross Site Scripting | Tampering | Mitigation Implemented |
| 244 | 165 | 5 | User entries and group memberships | Potential Data Repudiation by Synchronize federated users | Repudiation | Needs Investigation |
| 245 | 166 | 5 | User entries and group memberships | Data Flow Sniffing | Information Disclosure | Mitigation Implemented |
| 246 | 167 | 5 | User entries and group memberships | Potential Process Crash or Stop for Synchronize federated users | Denial Of Service | Needs Investigation |
| 247 | 168 | 5 | User entries and group memberships | Data Flow User entries and group memberships Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 248 | 169 | 5 | User entries and group memberships | Elevation Using Impersonation | Elevation Of Privilege | Needs Investigation |
| 249 | 170 | 5 | User entries and group memberships | Synchronize federated users May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 250 | 171 | 5 | User entries and group memberships | Elevation by Changing the Execution Flow in Synchronize federated users | Elevation Of Privilege | Mitigation Implemented |
| 251 | 100 | 5 | Imported users and role mappings | Spoofing of Destination Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 252 | 101 | 5 | Imported users and role mappings | Potential SQL Injection Vulnerability for Keycloak database | Tampering | Mitigation Implemented |
| 253 | 102 | 5 | Imported users and role mappings | Potential Excessive Resource Consumption for Synchronize federated users or Keycloak database | Denial Of Service | Mitigation Implemented |
| 254 | 89 | 5 | Federation configuration and mappers | Spoofing of Source Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 255 | 90 | 5 | Federation configuration and mappers | Cross Site Scripting | Tampering | Not Applicable |
| 256 | 91 | 5 | Federation configuration and mappers | Persistent Cross Site Scripting | Tampering | Not Applicable |
| 257 | 92 | 5 | Federation configuration and mappers | Weak Access Control for a Resource | Information Disclosure | Mitigation Implemented |
| 258 | 93 | 5 | Federation events | Spoofing of Destination Data Store Event store | Spoofing | Mitigation Implemented |
| 259 | 94 | 5 | Federation events | Potential SQL Injection Vulnerability for Event store | Tampering | Mitigation Implemented |
| 260 | 95 | 5 | Federation events | Potential Excessive Resource Consumption for Synchronize federated users or Event store | Denial Of Service | Mitigation Implemented |
| 261 | 188 | 5 | Validated user and federated roles | Synchronize federated users Process Memory Tampered | Tampering | Not Applicable |
| 262 | 189 | 5 | Validated user and federated roles | Cross Site Scripting | Tampering | Mitigation Implemented |
| 263 | 190 | 5 | Validated user and federated roles | Elevation Using Impersonation | Elevation Of Privilege | Needs Investigation |
| 264 | 119 | 2 | Registration, password reset and account requests | Spoofing the End user (browser) External Entity | Spoofing | Mitigation Implemented |
| 265 | 120 | 2 | Registration, password reset and account requests | Cross Site Scripting | Tampering | Mitigation Implemented |
| 266 | 121 | 2 | Registration, password reset and account requests | Potential Data Repudiation by Manage user account (register, reset password, update) | Repudiation | Mitigation Implemented |
| 267 | 122 | 2 | Registration, password reset and account requests | Potential Process Crash or Stop for Manage user account (register, reset password, update) | Denial Of Service | Mitigation Implemented |
| 268 | 123 | 2 | Registration, password reset and account requests | Data Flow Registration, password reset and account requests Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 269 | 124 | 2 | Registration, password reset and account requests | Elevation Using Impersonation | Elevation Of Privilege | Mitigation Implemented |
| 270 | 125 | 2 | Registration, password reset and account requests | Manage user account (register, reset password, update) May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 271 | 126 | 2 | Registration, password reset and account requests | Elevation by Changing the Execution Flow in Manage user account (register, reset password, update) | Elevation Of Privilege | Mitigation Implemented |
| 272 | 127 | 2 | Registration, password reset and account requests | Cross Site Request Forgery | Elevation Of Privilege | Mitigation Implemented |
| 273 | 4 | 2 | Account pages and responses | Spoofing of the End user (browser) External Destination Entity | Spoofing | Mitigation Implemented |
| 274 | 5 | 2 | Account pages and responses | External Entity End user (browser) Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 275 | 6 | 2 | Account pages and responses | Data Flow Account pages and responses Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 276 | 172 | 2 | User profile and policies | Spoofing of Source Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 277 | 173 | 2 | User profile and policies | Cross Site Scripting | Tampering | Mitigation Implemented |
| 278 | 174 | 2 | User profile and policies | Persistent Cross Site Scripting | Tampering | Mitigation Implemented |
| 279 | 175 | 2 | User profile and policies | Weak Access Control for a Resource | Information Disclosure | Mitigation Implemented |
| 280 | 154 | 2 | User account changes | Spoofing of Destination Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 281 | 155 | 2 | User account changes | Potential SQL Injection Vulnerability for Keycloak database | Tampering | Mitigation Implemented |
| 282 | 156 | 2 | User account changes | Potential Excessive Resource Consumption for Manage user account (register, reset password, update) or Keycloak database | Denial Of Service | Mitigation Implemented |
| 283 | 191 | 2 | Verification and reset emails | Spoofing of the Email service External Destination Entity | Spoofing | Mitigation Implemented |
| 284 | 192 | 2 | Verification and reset emails | External Entity Email service Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 285 | 193 | 2 | Verification and reset emails | Data Flow Verification and reset emails Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 286 | 1 | 2 | Account events | Spoofing of Destination Data Store Event store | Spoofing | Mitigation Implemented |
| 287 | 2 | 2 | Account events | Potential SQL Injection Vulnerability for Event store | Tampering | Mitigation Implemented |
| 288 | 3 | 2 | Account events | Potential Excessive Resource Consumption for Manage user account (register, reset password, update) or Event store | Denial Of Service | Mitigation Implemented |
| 289 | 14 | 2 | Admin requests | Spoofing the Realm admin (browser) External Entity | Spoofing | Mitigation Implemented |
| 290 | 15 | 2 | Admin requests | Cross Site Scripting | Tampering | Mitigation Implemented |
| 291 | 16 | 2 | Admin requests | Potential Data Repudiation by Administer realm (Admin console) | Repudiation | Mitigation Implemented |
| 292 | 17 | 2 | Admin requests | Potential Process Crash or Stop for Administer realm (Admin console) | Denial Of Service | Mitigation Implemented |
| 293 | 18 | 2 | Admin requests | Data Flow Admin requests Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 294 | 19 | 2 | Admin requests | Elevation Using Impersonation | Elevation Of Privilege | Needs Investigation |
| 295 | 20 | 2 | Admin requests | Administer realm (Admin console) May be Subject to Elevation of Privilege Using Remote Code Execution | Elevation Of Privilege | Mitigation Implemented |
| 296 | 21 | 2 | Admin requests | Elevation by Changing the Execution Flow in Administer realm (Admin console) | Elevation Of Privilege | Mitigation Implemented |
| 297 | 22 | 2 | Admin requests | Cross Site Request Forgery | Elevation Of Privilege | Mitigation Implemented |
| 298 | 23 | 2 | Admin responses | Spoofing of the Realm admin (browser) External Destination Entity | Spoofing | Mitigation Implemented |
| 299 | 24 | 2 | Admin responses | External Entity Realm admin (browser) Potentially Denies Receiving Data | Repudiation | Mitigation Implemented |
| 300 | 25 | 2 | Admin responses | Data Flow Admin responses Is Potentially Interrupted | Denial Of Service | Mitigation Implemented |
| 301 | 10 | 2 | Admin permissions and realm configuration | Spoofing of Source Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 302 | 11 | 2 | Admin permissions and realm configuration | Cross Site Scripting | Tampering | Mitigation Implemented |
| 303 | 12 | 2 | Admin permissions and realm configuration | Persistent Cross Site Scripting | Tampering | Mitigation Implemented |
| 304 | 13 | 2 | Admin permissions and realm configuration | Weak Access Control for a Resource | Information Disclosure | Mitigation Implemented |
| 305 | 176 | 2 | User, role, client and flow changes | Spoofing of Destination Data Store Keycloak database | Spoofing | Mitigation Implemented |
| 306 | 177 | 2 | User, role, client and flow changes | Potential SQL Injection Vulnerability for Keycloak database | Tampering | Mitigation Implemented |
| 307 | 178 | 2 | User, role, client and flow changes | Potential Excessive Resource Consumption for Administer realm (Admin console) or Keycloak database | Denial Of Service | Mitigation Implemented |
| 308 | 7 | 2 | Admin events | Spoofing of Destination Data Store Event store | Spoofing | Mitigation Implemented |
| 309 | 8 | 2 | Admin events | Potential SQL Injection Vulnerability for Event store | Tampering | Mitigation Implemented |
| 310 | 9 | 2 | Admin events | Potential Excessive Resource Consumption for Administer realm (Admin console) or Event store | Denial Of Service | Mitigation Implemented |
