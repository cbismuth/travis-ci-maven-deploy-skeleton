# How to deploy to Maven Central by Travis-CI

``` bash
git clone https://github.com/cbismuth/travis-ci-maven-deploy-skeleton.git
cd travis-ci-maven-deploy-skeleton
cp -R deploy <path to your project>/
cp .travis.yml <path to your project>/
cd <path to your project>

gem install travis
travis login
travis enable -r $(echo $USER)/${PWD##*/}

export ENCRYPTION_PASSWORD=<password to encrypt>
openssl aes-256-cbc -pass pass:$ENCRYPTION_PASSWORD -in ~/.gnupg/secring.gpg -out deploy/secring.gpg.enc
openssl aes-256-cbc -pass pass:$ENCRYPTION_PASSWORD -in ~/.gnupg/pubring.gpg -out deploy/pubring.gpg.enc

travis encrypt --add -r $(echo $USER)/${PWD##*/} SONATYPE_USERNAME=cbismuth
travis encrypt --add -r $(echo $USER)/${PWD##*/} SONATYPE_PASSWORD=<sonatype password>
travis encrypt --add -r $(echo $USER)/${PWD##*/} ENCRYPTION_PASSWORD=$ENCRYPTION_PASSWORD
travis encrypt --add -r $(echo $USER)/${PWD##*/} GPG_KEYNAME=D7E158F1
travis encrypt --add -r $(echo $USER)/${PWD##*/} GPG_PASSPHRASE=$ENCRYPTION_PASSWORD
```

Add the following elements in your Maven POM descriptor.

``` xml
    <distributionManagement>
        <snapshotRepository>
            <id>ossrh</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
        <repository>
            <id>ossrh</id>
            <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>
```

``` xml
    <profiles>
        <profile>
            <id>ossrh</id>
            <properties>
                <gpg.executable>gpg</gpg.executable>
                <gpg.keyname>${env.GPG_KEYNAME}</gpg.keyname>
                <gpg.passphrase>${env.GPG_PASSPHRASE}</gpg.passphrase>
                <gpg.defaultKeyring>false</gpg.defaultKeyring>
                <gpg.publicKeyring>${env.DEPLOY_DIR}/pubring.gpg</gpg.publicKeyring>
                <gpg.secretKeyring>${env.DEPLOY_DIR}/secring.gpg</gpg.secretKeyring>
            </properties>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>${maven-source-plugin.version}</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>

                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>${maven-javadoc-plugin.version}</version>
                        <executions>
                            <execution>
                                <id>attach-javadocs</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>

                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>${maven-gpg-plugin.version}</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>

                    <plugin>
                        <groupId>org.sonatype.plugins</groupId>
                        <artifactId>nexus-staging-maven-plugin</artifactId>
                        <version>${nexus-staging-maven-plugin.version}</version>
                        <extensions>true</extensions>
                        <configuration>
                            <serverId>ossrh</serverId>
                            <nexusUrl>https://oss.sonatype.org/</nexusUrl>
                            <autoReleaseAfterClose>true</autoReleaseAfterClose>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
```
