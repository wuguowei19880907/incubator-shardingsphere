+++
title = "Release Guide"
weight = 6
chapter = true
+++

## GPG Settings

### Install GPG

Download installation package on [official GnuPG website](https://www.gnupg.org/download/index.html). 
The command of GnuPG 1.x version can differ a little from that of 2.x version. 
The following instructions take `GnuPG-2.1.23` version for example.
After the installation, execute the following command to check the version number.

```shell
gpg --version
```

### Create Key

After the installation, execute the following command to create key.

This command indicates `GnuPG-2.x` can be used:

```shell
gpg --full-gen-key
```

This command indicates `GnuPG-1.x` can be used:

```shell
gpg --gen-key
```

Finish the key creation according to instructions:

**Notice: Please use Apache mail for key creation.**

```shell
gpg (GnuPG) 2.0.12; Copyright (C) 2009 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
  (1) RSA and RSA (default)
  (2) DSA and Elgamal
  (3) DSA (sign only)
  (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
        0 = key does not expire
     <n>  = key expires in n days
     <n>w = key expires in n weeks
     <n>m = key expires in n months
     <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: ${Input username}
Email address: ${Input email}
Comment: ${Input comment}
You selected this USER-ID:
   "${Inputed username} (${Inputed comment}) <${Inputed email}>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key. # Input passwords
```

### Check Generated Key

```shell
gpg --list-keys
```

Execution Result:

```shell
pub   4096R/700E6065 2019-03-20
uid                  ${Username} (${Comment}) <{Email}>
sub   4096R/0B7EF5B2 2019-03-20
```

Among them, 700E6065 is public key ID.

### Upload the Public Key to Key Server

The command is as follow:

```shell
gpg --keyserver hkp://pool.sks-keyservers.net --send-key 700E6065
```

`pool.sks-keyservers.net` is randomly chosen from [public key server](https://sks-keyservers.net/status/). 
Each server will automatically synchronize with one another, so it would be okay to choose any one.

## Apache Maven Central Repository Release

### Set settings.xml

Add the following template to `~/.m2/settings.xml`, all the passwords need to be filled in after encryption. 
For encryption settings, please see [here](http://maven.apache.org/guides/mini/guide-encryption.html).

```xml
<settings>
    <servers>
      <server>
          <id>apache.snapshots.https</id>
          <username> <!-- APACHE LDAP username --> </username>
          <password> <!-- APACHE LDAP encrypted password --> </password>
      </server>
      <server>
          <id>apache.releases.https</id>
          <username> <!-- APACHE LDAP username --> </username>
          <password> <!-- APACHE LDAP encrypted password --> </password>
      </server>
    </servers>
</settings>
```

### Update Release Notes

```
https://github.com/apache/incubator-shardingsphere/blob/master/RELEASE-NOTES.md
```

### Create Release Branch

Suppose ShardingSphere source codes downloaded from github is under `~/incubator-shardingsphere/` directory and the version to be released is `4.0.0-RC`. 
Create `${RELEASE.VERSION}-release` branch, where all the following operations are performed.

```shell
## ${name} is the properly branch, e.g. master, dev-4.x
git clone --branch ${name} https://github.com/apache/incubator-shardingsphere.git ~/incubator-shardingsphere
cd ~/incubator-shardingsphere/
git pull
git checkout -b ${RELEASE.VERSION}-release
git push origin ${RELEASE.VERSION}-release
```

### Pre-Release Check

```shell
mvn release:prepare -Prelease -Darguments="-DskipTests" -DautoVersionSubmodules=true -DdryRun=true -Dusername=${Github username}
```

-Prelease: choose release profile, which will pack all the source codes, jar files and executable binary packages of sharding-proxy.

-DautoVersionSubmodules=true: it can make the version number is inputted only once and not for each sub-module.

-DdryRun=true: rehearsal, which means not to generate or submit new version number and new tag.

### Prepare for the Release

First, clean local pre-release check information.

```shell
mvn release:clean
```

Then, prepare to execute the release.

```shell
mvn release:prepare -Prelease -Darguments="-DskipTests" -DautoVersionSubmodules=true -DpushChanges=false -Dusername=${Github username}
```

It is basically the same as the previous rehearsal command, but deleting -DdryRun=true parameter.

-DpushChanges=false: do not submit the edited version number and tag to Github automatically.

After making sure there is no mistake in local files, submit them to GitHub.

```shell
git push
git push origin --tags
```

### Deploy the Release

```shell
mvn release:perform -Prelease -Darguments="-DskipTests" -DautoVersionSubmodules=true -Dusername=${Github username}
```

After that command is executed, the version to be released will be uploaded to Apache staging repository automatically. 
Visit [https://repository.apache.org/#stagingRepositories](https://repository.apache.org/#stagingRepositories) and use Apache LDAP account to log in; then you can see the uploaded version, the content of `Repository` column is the ${STAGING.REPOSITORY}. 
Click `Close` to tell Nexus that the construction is finished, because only in this way, this version can be usable. 
If there is any problem in gpg signature, `Close` will fail, but you can see the failure information through `Activity`.

## Apache SVN Repository Release

### Checkout ShardingSphere Release Directory

If there is no local work directory, create one at first.

```shell
mkdir -p ~/ss_svn/dev/
cd ~/ss_svn/dev/
```

After the creation, checkout ShardingSphere release directory from Apache SVN.

```shell
svn --username=${APACHE LDAP username} co https://dist.apache.org/repos/dist/dev/incubator/shardingsphere
cd ~/ss_svn/dev/shardingsphere
```

### Add gpg Public Key

Only the account in its first deployment needs to add that. 
It is alright for `KEYS` to only include the public key of the deployed account.

```shell
gpg -a --export ${GPG username} >> KEYS
```

### Add the Release Content to SVN Directory

Create folder by version number.

```shell
mkdir -p ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cd ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
```

Add source code packages, binary packages and executable binary packages of sharding-proxy to SVN working directory.

```shell
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/shardingsphere-src-distribution/target/*.zip ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/shardingsphere-src-distribution/target/*.zip.asc ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-jdbc-distribution/target/*.tar.gz ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-jdbc-distribution/target/*.tar.gz.asc ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-proxy-distribution/target/*.tar.gz ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-proxy-distribution/target/*.tar.gz.asc ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-scaling-distribution/target/*.tar.gz ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-scaling-distribution/target/*.tar.gz.asc ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-ui-distribution/target/*.tar.gz ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
cp -f ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-ui-distribution/target/*.tar.gz.asc ~/ss_svn/dev/shardingsphere/${RELEASE.VERSION}
```

### Generate sign files

```shell
shasum -a 512 apache-shardingsphere-incubating-${RELEASE.VERSION}-src.zip >> apache-shardingsphere-incubating-${RELEASE.VERSION}-src.zip.sha512
shasum -b -a 512 apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-jdbc-bin.tar.gz >> apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-jdbc-bin.tar.gz.sha512
shasum -b -a 512 apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-proxy-bin.tar.gz >> apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-proxy-bin.tar.gz.sha512
shasum -b -a 512 apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-scaling-bin.tar.gz >> apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-scaling-bin.tar.gz.sha512
shasum -b -a 512 apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-ui-bin.tar.gz >> apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-ui-bin.tar.gz.sha512
```

### Commit to Apache SVN

```shell
svn add *
svn --username=${APACHE LDAP username} commit -m "release ${RELEASE.VERSION}"
```

## Check Release

### Check sha512 hash

```shell
shasum -c apache-shardingsphere-incubating-${RELEASE.VERSION}-src.zip.sha512
shasum -c apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-jdbc-bin.tar.gz.sha512
shasum -c apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-proxy-bin.tar.gz.sha512
shasum -c apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-scaling-bin.tar.gz.sha512
shasum -c apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-ui-bin.tar.gz.sha512
```

### Check gpg Signature

First, import releaser's public key. 
Import KEYS from SVN repository to local. (The releaser does not need to import again; the checking assistant needs to import it, with the user name filled as the releaser's. )

```shell
curl https://dist.apache.org/repos/dist/dev/incubator/shardingsphere/KEYS >> KEYS
gpg --import KEYS
gpg --edit-key "${GPG username of releaser}"
  > trust

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5

  > save
```

Then, check the gpg signature.

```shell
gpg --verify apache-shardingsphere-incubating-${RELEASE.VERSION}-src.zip.asc apache-shardingsphere-incubating-${RELEASE.VERSION}-src.zip
gpg --verify apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-jdbc-bin.tar.gz.asc apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-jdbc-bin.tar.gz
gpg --verify apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-proxy-bin.tar.gz.asc apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-proxy-bin.tar.gz
gpg --verify apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-scaling-bin.tar.gz.asc apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-scaling-bin.tar.gz
gpg --verify apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-ui-bin.tar.gz.asc apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-ui-bin.tar.gz
```

### Check Released Files

#### compare release source with github tag

```
curl -Lo tag-${RELEASE.VERSION}.zip https://github.com/apache/incubator-shardingsphere/archive/${RELEASE.VERSION}.zip | unzip
unzip apache-shardingsphere-incubating-${RELEASE.VERSION}-src.zip
diff -r apache-shardingsphere-incubating-${RELEASE.VERSION}-src tag-${RELEASE.VERSION}
```

#### Check source package

*   Check whether source tarball is oversized for including nonessential files
*   The release files have the word `incubating` in their name
*   `DISCLAIMER` file exists
*   `LICENSE` and `NOTICE` files exist
*   Correct year in `NOTICE` file
*   There is only text files but no binary files
*   All source files have ASF headers
*   Codes can be compiled and pass the unit tests (./mvnw install)
*   Check if there is any extra files or folders, empty folders for example

#### Check binary packages

Decompress `apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-jdbc-bin.tar.gz`, `apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-proxy-bin.tar.gz` ,
`apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-scaling-bin.tar.gz` and `apache-shardingsphere-incubating-${RELEASE.VERSION}-sharding-ui-bin.tar.gz`
to check the following items:

*   The release files have the word `incubating` in their name
*   `DISCLAIMER` file exists
*   `LICENSE` and `NOTICE` files exist
*   Correct year in `NOTICE` file
*   All text files have ASF headers
*   Check the third party dependency license:
    *   The software have a compatible license
    *   All software licenses mentioned in `LICENSE`
    *   All the third party dependency licenses are under `licenses` folder
    *   If it depends on Apache license and has a `NOTICE` file, that `NOTICE` file need to be added to `NOTICE` file of the release

For the whole check list, please see [here](https://cwiki.apache.org/confluence/display/INCUBATOR/Incubator+Release+Checklist).

## Call for a Vote

### Vote procedure

1. ShardingSphere community vote: send the vote e-mail to `dev@shardingsphere.apache.org`. 
PPMC needs to check the rightness of the version according to the document before they vote. 
After at least 72 hours and with at least 3 `+1 PPMC member` votes, it can come to the next stage of the vote.

2. Apache community vote: send the vote e-mail to `general@incubator.apache.org`.
After at least 72 hours and with at least 3 `+1 binding` votes (only IPMC's votes are binding), it can be officially released.

3. Announce the vote result: send the result vote e-mail to `general@incubator.apache.org`.

### Vote Templates

1. ShardingSphere Community Vote Template

NOTE: Must invite all mentors to vote during the community vote.

Title:

```
[VOTE] Release Apache ShardingSphere (Incubating) ${RELEASE.VERSION}

```

Body:

```
Hello ShardingSphere Community,

This is a call for vote to release Apache ShardingSphere (Incubating) version ${RELEASE.VERSION}

Release notes:
https://github.com/apache/incubator-shardingsphere/blob/master/RELEASE-NOTES.md

The release candidates:
https://dist.apache.org/repos/dist/dev/incubator/shardingsphere/${RELEASE.VERSION}/

Maven 2 staging repository:
https://repository.apache.org/content/repositories/${STAGING.REPOSITORY}/org/apache/shardingsphere/

Git tag for the release:
https://github.com/apache/incubator-shardingsphere/tree/${RELEASE.VERSION}

Release Commit ID:
https://github.com/apache/incubator-shardingsphere/commit/xxxxxxxxxxxxxxxxxxxxxxx

Keys to verify the Release Candidate:
https://dist.apache.org/repos/dist/dev/incubator/shardingsphere/KEYS

Look at here for how to verify this release candidate:
https://shardingsphere.apache.org/community/en/contribute/release/

The vote will be open for at least 72 hours or until necessary number of votes are reached.

Please vote accordingly:

[ ] +1 approve 

[ ] +0 no opinion
 
[ ] -1 disapprove with the reason

Checklist for reference:

[ ] Download links are valid.

[ ] Checksums and PGP signatures are valid.

[ ] DISCLAIMER is included.

[ ] Source code distributions have correct names matching the current release.

[ ] LICENSE and NOTICE files are correct for each ShardingSphere repo.

[ ] All files have license headers if necessary.

[ ] No compiled archives bundled in source archive.
```

2. Announce the vote result:

Body:

```
The vote to release Apache ShardingSphere (Incubating) ${RELEASE.VERSION} has passed.

7 PPMC member +1 votes:

xxx (mentor)
xxx
xxx (mentor)
xxx
xxx
xxx (mentor)
xxx

1 community +1 vote:
xxx

Thank you everyone for taking the time to review the release and help us. 
```

3. Apache Community Vote Template:

Title:

```
[VOTE] Release Apache ShardingSphere (Incubating) ${RELEASE.VERSION}

```

Body:

```
Hello all,

This is a call for vote to release Apache ShardingSphere (Incubating) version ${RELEASE.VERSION}.

The Apache ShardingSphere community has voted on and approved a proposal to release
Apache ShardingSphere (Incubating) version ${RELEASE.VERSION}.

We now kindly request the Incubator PMC members review and vote on this
incubator release.

ShardingSphere is an open-source ecosystem consisted of a set of distributed database middleware solutions, including 2 independent products, Sharding-JDBC & Sharding-Proxy. 
They both provide functions of data sharding, distributed transaction and database orchestration, applicable in a variety of situations such as Java isomorphism, heterogeneous language. 
Aiming at reasonably making full use of the computation and storage capacity of the database in a distributed system, ShardingSphere defines itself as a middleware, rather than a totally new type of database. 
As the cornerstone of many enterprises, relational database still takes a huge market share. 
Therefore, at the current stage, we prefer to focus on its increment instead of a total overturn.

Sharding-JDBC defines itself as a lightweight Java framework that provides extra service at Java JDBC layer. 
With the client end connecting directly to the database, it provides service in the form of jar and requires no extra deployment and dependence. 
It can be considered as an enhanced JDBC driver, which is fully compatible with JDBC and all kinds of ORM frameworks.

* Applicable in any ORM framework based on Java, such as JPA, Hibernate, Mybatis, Spring JDBC Template or direct use of JDBC.
* Based on any third-party database connection pool, such as DBCP, C3P0, BoneCP, Druid, HikariCP.
* Support any kind of database that conforms to JDBC standard: MySQL, Oracle, SQLServer and PostgreSQL for now.

Sharding-Proxy defines itself as a transparent database proxy, providing a database server that encapsulates database binary protocol to support heterogeneous languages. 
Friendlier to DBA, the MySQL/PostgreSQL version provided now can use any kind of terminal (such as MySQL Command Client, MySQL Workbench, Navicat etc.) that is compatible of MySQL/PostgreSQL protocol to operate data.

* Totally transparent to applications, it can be used directly as MySQL and PostgreSQL.

* Applicable to any kind of terminal that is compatible with MySQL and PostgreSQL protocol.

ShardingSphere community vote and result thread:
https://lists.apache.org/thread.html/xxxxxxxxxxxxxxxxxxxxxxx

Release notes:
https://github.com/apache/incubator-shardingsphere/blob/master/RELEASE-NOTES.md

The release candidates:
https://dist.apache.org/repos/dist/dev/incubator/shardingsphere/${RELEASE.VERSION}/

Maven 2 staging repository:
https://repository.apache.org/content/repositories/${STAGING.REPOSITORY}/org/apache/shardingsphere/

Git tag for the release:
https://github.com/apache/incubator-shardingsphere/tree/${RELEASE.VERSION}

Release Commit ID:
https://github.com/apache/incubator-shardingsphere/commit/xxxxxxxxxxxxxxxxxxxxxxx

Keys to verify the Release Candidate:
https://dist.apache.org/repos/dist/dev/incubator/shardingsphere/KEYS

Look at here for how to verify this release candidate:
https://shardingsphere.apache.org/community/en/contribute/release/

The vote will be open for at least 72 hours or until necessary number of
votes are reached.

Please vote accordingly:

[ ] +1 approve

[ ] +0 no opinion

[ ] -1 disapprove with the reason

Checklist for reference:

[ ] Download links are valid.

[ ] Checksums and PGP signatures are valid.

[ ] DISCLAIMER is included.

[ ] Source code distributions have correct names matching the current release.

[ ] LICENSE and NOTICE files are correct for each ShardingSphere repo.

[ ] All files have license headers if necessary.

[ ] No compiled archives bundled in source archive.

The following votes are carried over from ShardingSphere dev mailing list,

+1 binding, xxx
+1 binding, xxx

+1 non-binding, xxx
+1 non-binding, xxx

```

4. Announce the vote result:

**Notice: Please include the votes from ShardingSphere community above.**

Title：

```
[RESULT][VOTE] Release Apache ShardingSphere (Incubating) ${RELEASE.VERSION}
```

Body:

```
We’ve received 3 +1 binding votes and one +1 non-binding vote:

+1 binding, xxx
+1 binding, xxx
+1 binding, xxx

+1 non-binding, xxx

Thank you everyone for taking the time to review the release and help us. 
I will process to publish the release and send ANNOUNCE.

```

## Finish the Release

### Move source packages, binary packages and KEYS from the `dev` directory to `release` directory

```shell
svn mv https://dist.apache.org/repos/dist/dev/incubator/shardingsphere/${RELEASE.VERSION} https://dist.apache.org/repos/dist/release/incubator/shardingsphere/ -m "transfer packages for ${RELEASE.VERSION}"
svn delete https://dist.apache.org/repos/dist/release/incubator/shardingsphere/KEYS -m "delete KEYS"
svn cp https://dist.apache.org/repos/dist/dev/incubator/shardingsphere/KEYS https://dist.apache.org/repos/dist/release/incubator/shardingsphere/ -m "transfer KEYS for ${RELEASE.VERSION}"
```

### Find ShardingSphere in staging repository and click `Release`

### Merge release branch to `master` and delete release branch on Github

```shell
git checkout master
git merge origin/${RELEASE.VERSION}-release
git push
git push --delete origin ${RELEASE.VERSION}-release
```

### Update READEME files

Update `${RELEASE.VERSION}` to `${NEXT.RELEASE.VERSION}` in README.md and README_ZH.md

Update `${RELEASE.VERSION}` to `${NEXT.RELEASE.VERSION}` for `CURRENT_VERSION` in `Dockerfile`

Update `${RELEASE.VERSION}` to `${NEXT.RELEASE.VERSION}` for `imageName` in Maven `Docker` plugin

Update `${RELEASE.VERSION}` to `${NEXT.RELEASE.VERSION}` for `SERVER_VERSION` in `MySQLServerInfo.java`

### Update the download page

https://shardingsphere.apache.org/document/current/en/downloads/

https://shardingsphere.apache.org/document/current/cn/downloads/

Keep two latest versions in `Latest releases`. All old versions will be archived automatically in [Archive incubating repository](https://archive.apache.org/dist/incubator/shardingsphere/)

### Docker Release

#### Preparation

Install docker locally and start the docker service

#### Compile Docker Image

```shell
cd ~/shardingsphere/incubator-shardingsphere/sharding-distribution/sharding-proxy-distribution/
mvn clean package docker:build
```

#### Tag the local Docker Image

Check the image ID through `docker images`, for example: e9ea51023687

```shell
docker tag e9ea51023687 apache/sharding-proxy:latest
docker tag e9ea51023687 apache/sharding-proxy:${RELEASE.VERSION}
```

#### Publish Docker Image

```shell
docker push apache/sharding-proxy:latest
docker push apache/sharding-proxy:${RELEASE_VERSION}
```

#### Confirm the successful release

Login [Docker Hub](https://hub.docker.com/r/apache/sharding-proxy/) to check whether there are published images

### Publish release in GitHub

Click `Edit` in [GitHub Releases](https://github.com/apache/incubator-shardingsphere/releases)'s `${RELEASE_VERSION}` version

Edit version number and release notes, click `Publish release`

### Tag ShardingSphere's Example Project

[GitHub repository](https://github.com/apache/incubator-shardingsphere-example)

### Send e-mail to `general@incubator.apache.org` and `dev@shardingsphere.apache.org` to announce the release is finished

Announcement e-mail template:

Title:

```
[ANNOUNCE] Apache ShardingSphere ${RELEASE.VERSION} available
```

Body:

```
Hi all,

Apache ShardingSphere (incubating) Team is glad to announce the first release of Apache ShardingSphere Incubating ${RELEASE.VERSION}.

ShardingSphere is an open-source ecosystem consisted of a set of distributed database middleware solutions, including 2 independent products, Sharding-JDBC & Sharding-Proxy. 
They both provide functions of data sharding, distributed transaction and database orchestration, applicable in a variety of situations such as Java isomorphism, heterogeneous language. 
Aiming at reasonably making full use of the computation and storage capacity of the database in a distributed system, ShardingSphere defines itself as a middleware, rather than a totally new type of database. 
As the cornerstone of many enterprises, relational database still takes a huge market share. 
Therefore, at the current stage, we prefer to focus on its increment instead of a total overturn.

Download Links: https://shardingsphere.apache.org/document/current/en/downloads/

Release Notes: https://github.com/apache/incubator-shardingsphere/blob/master/RELEASE-NOTES.md

Website: https://shardingsphere.apache.org/

ShardingSphere Resources:
- Issue: https://github.com/apache/incubator-shardingsphere/issues/
- Mailing list: dev@shardingsphere.apache.org
- Documents: https://shardingsphere.apache.org/document/current/



- Apache ShardingSphere (incubating) Team


=====
*Disclaimer*

Apache ShardingSphere (incubating) is an effort undergoing incubation at The Apache Software Foundation (ASF), sponsored by the Apache Incubator PMC.
Incubation is required of all newly accepted projects until a further review indicates that the infrastructure, 
communications, and decision making process have stabilized in a manner consistent with other successful ASF projects. 
While incubation status is not necessarily a reflection of the completeness or stability of the code, 
it does indicate that the project has yet to be fully endorsed by the ASF.

```
