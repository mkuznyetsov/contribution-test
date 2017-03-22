# DTO

- [What is DTO?](#what-is-dto)
- [Prepare interfaces](#prepare-interfaces)
- [Configure pom.xml](#configure)
- [Generating DTO](#generating-dto)
- [How to use generated DTOs](#how-to-use-generated-dtos)
- [Method chaining](#method-chaining)
- [Delegate DTO methods' call](#delegate-dto-methods-call)


## What is DTO?

DTO (Data transfer object) - is an object to exchange data with the client and server. DTOs are responsible for serializing themselves into JSON and vise versa (deserializing raw data into DTO).

## Prepare interfaces

First, prepare interfaces, that must contains get-methods at least. Mark these interfaces with the org.eclipse.che.dto.shared.DTO annotation.
Possible names for getter-methods are:
- get...
- is...

For example:
```
package com.codenvy.test.dto;

import org.eclipse.che.dto.shared.DTO;

@DTO
public interface Item {
    String getStatus();
    void setStatus(String status);
    boolean isHidden();
    void setHidden(boolean hidden);
}
```
## Configure pom.xml

Next, plugins section in pom.xml should be properly configured.

1. maven-compiler-plugin - helps to run compilation of project in two steps.
2. che-core-api-dto-maven-plugin - generates DTO implementations.
3. build-helper-maven-plugin - helps to add generated (by che-core-api-dto-maven-plugin) files to project's sources.

**che-core-api-dto-maven-plugin** has the following configuration parameters (all are required):

*dtoPackages* - names of packages which contains DTO interfaces.

*outputDirectory* - name of directory to store generated files. Usually the best way set one property, e.g. dto-generator-out-directory and reuse it in
configurations of plugins (see code snippet below).

*genClassName* - all generated classes are placed in single enclosing class. This parameter sets name of that class.

*impl* - which DTO generate. Supported two types: server or client.
dependencies - each maven plugin has own classloader. Plugin cannot see dependencies or build output of current project. So it's necessary to
add current project as dependency for che-core-api-dto-maven-plugin. If DTOs from current project use DTOs from another project than need add
dependency to that project too.

The following pom.xml snippet demonstrates how you can generate server-side DTOs:
```
...
<properties>
    <dto-generator-out-directory>${project.build.directory}/generated-sources/dto/</dto-generator-out-directory>
</properties>
....
<build>
<plugins>
    <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <executions>
            <execution>
                <id>pre-compile</id>
                <phase>generate-sources</phase>
                <goals>
                    <goal>compile</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    <plugin>
        <groupId>org.eclipse.che.core</groupId>
        <artifactId>che-core-api-dto-maven-plugin</artifactId>
        <version>${che.core.version}</version>
        <executions>
            <execution>
                <phase>process-sources</phase>
                <goals>
                    <goal>generate</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <dtoPackages>
                <package>com.codenvy.test.dto</package>
            </dtoPackages>

            <outputDirectory>${dto-generator-out-directory}</outputDirectory>

            <genClassName>com.codenvy.test.server.dto.DtoServerImpls</genClassName>
            <impl>server</impl>
        </configuration>
        <dependencies>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>${project.artifactId}</artifactId>
                <version>${project.version}</version>
            </dependency>
            <!--
            Other dependencies if DTOs from current project need them.
            -->
        </dependencies>
    </plugin>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <executions>
            <execution>
                <id>add-resource</id>
                <phase>process-sources</phase>
                <goals>
                    <goal>add-resource</goal>
                </goals>
                <configuration>
                    <resources>
                        <resource>

                            <directory>${dto-generator-out-directory}/META-INF</directory>
                            <targetPath>META-INF</targetPath>
                        </resource>
                    </resources>
                </configuration>
            </execution>
            <execution>
                <id>add-source</id>
                <phase>process-sources</phase>
                <goals>
                    <goal>add-source</goal>
                </goals>
                <configuration>
                    <sources>
                        <source>${dto-generator-out-directory}</source>
                    </sources>
                </configuration>
            </execution>
        </executions>
    </plugin>
</plugins>
</build>
...
```

The following pom.xml snippet demonstrates how you can generate both types of DTOs (client and server):
```
...
...
<properties>
    <dto-generator-out-directory>${project.build.directory}/generated-sources/dto/</dtogenerator-out-directory>
</properties>
...
<build>
<plugins>
    <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <executions>
            <execution>
                <id>pre-compile</id>
                <phase>generate-sources</phase>
                <goals>
                    <goal>compile</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    <plugin>
        <groupId>org.eclipse.che.core</groupId>
        <artifactId>che-core-api-dto-maven-plugin</artifactId>
        <version>${che.core.version}</version>
        <executions>
            <execution>
                <phase>process-sources</phase>
                <goals>
                    <goal>generate</goal>
                </goals>
                <configuration>
                    <dtoPackages>
                        <package>com.codenvy.test.dto</package>
                    </dtoPackages>

                    <outputDirectory>${dto-generator-out-directory}</outputDirectory>

                    <genClassName>com.codenvy.test.client.dto.DtoClientImpls</genClassName>
                    <impl>client</impl>
                </configuration>
            </execution>
            <execution>
                <phase>process-sources</phase>
                <goals>
                    <goal>generate</goal>
                </goals>
                <configuration>
                    <dtoPackages>
                        <package>com.codenvy.test.dto</package>
                    </dtoPackages>

                    <outputDirectory>${dto-generator-out-directory}</outputDirectory>

                    <genClassName>com.codenvy.test.server.dto.DtoServerImpls</genClassName>
                    <impl>server</impl>
                </configuration>
            </execution>
        </executions>
        <dependencies>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>${project.artifactId}</artifactId>
                <version>${project.version}</version>
            </dependency>
            <!--
            Other dependencies if DTOs from current project need them.
            -->
        </dependencies>
    </plugin>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <executions>
            <execution>
                <id>add-resource</id>
                <phase>process-sources</phase>
                <goals>
                    <goal>add-resource</goal>
                </goals>
                <configuration>
                    <resources>
                        <resource>

                            <directory>${dto-generator-out-directory}/META-INF</directory>
                            <targetPath>META-INF</targetPath>
                        </resource>
                    </resources>
                </configuration>
            </execution>
            <execution>
                <id>add-source</id>
                <phase>process-sources</phase>
                <goals>
                    <goal>add-source</goal>
                </goals>
                <configuration>
                    <sources>
                        <source>${dto-generator-out-directory}</source>
                    </sources>
                </configuration>
            </execution>
        </executions>
    </plugin>
</plugins>
<resources>
    ...
    <resource>
        <directory>${generated.sources.directory}</directory>
    </resource>
</resources>
</build>
...
```

Make sure that the directory with generated client-side DTOs is included in classpath resources.

## Generating DTO

In order to generate DTOs, you should just build your project with Maven:
```
mvn clean install
```

### How to use generated DTOs

After the DTOs has been generated, you can use them into your project, on the server-side, as follows:
#### on the server-side
```

// server-side DTO
import org.eclipse.che.dto.server.DtoFactory;
...
// create instance and set fields
MyJob job = DtoFactory.getInstance().createDto(MyJob.class);
job.setStatus("success");
job.setExitCode(0);
// serialize to JSON
String json = DtoFactory.getInstance().toJson(job);
// deserialize from JSON
MyJob job2 = DtoFactory.getInstance().createDtoFromJson(json, MyJob.class);
```

> Tip
Also you can use the static method DtoFactory.newDto(MyJob.class) that is a shortcut for DtoFactory.getInstance().createDto(MyJob.cl
ass).

The following code snippet demonstrates how you can use generated DTOs on the client-side:

#### on the client side
```
// client-side DTO
import org.eclipse.che.ide.dto.DtoFactory;
@Singleton
public class MyPresenter {
    @Inject
    public MyPresenter(DtoFactory dtoFactory) {
        // create instance and set fields
        MyJob job = dtoFactory.createDto(MyJob.class);
        job.setStatus("success");
        job.setExitCode(0);
        // serialize to JSON
        String json = dtoFactory.toJson(job);
        // deserialize from JSON
        MyJob job2 = dtoFactory.createDtoFromJson(json, MyJob.class);
    }
}
```

## Method chaining

Additionally to getters and setters with are required for DTO, generator adds method DTO *withXXX(T value)*. Such method do the same as setter but it returns this. It makes possible to use chaining.
Instead of:
```
MyJob job = DtoFactory.getInstance().createDto(MyJob.class);
job.setStatus("success");
job.setExitCode(0);
...
```

use:
```
MyJob job =
DtoFactory.getInstance().createDto(MyJob.class).withStatus("success").withExitCode(0);
```

It may be used on the client-side in the same way.
Instead of:
```
...
MyJob job = dtoFactory.createDto(MyJob.class);
job.setStatus("success");
job.setExitCode(0);
...
```
use:
```
...
MyJob job = dtoFactory.createDto(MyJob.class).withStatus("success").withExitCode(0);
```

Generator always add such methods in generated implementation for your DTO interfaces (doesn't matter do you
declare them in interface or not). So if you want to use chaining for your DTOs you simply need add *withXXX(T
value*) methods in your DTO interfaces.
```
package com.codenvy.test.dto;
import org.eclipse.che.dto.shared.DTO;
@DTO
public interface MyJob {
     String getStatus();
     void setStatus(String status);
     int getExitCode();
     void setExitCode(int code);
     String getError();
     void setError(String error);
     // for chaining
     MyJob withStatus(String status);
     MyJob withExitCode(int code);
     MyJob withError(String error);
}
```
## Delegate DTO methods' call

In some case we may need more then just getters and setters in DTO, but there is no common mechanism to generate such implementation for DTO interface. In this case *org.eclipse.che.dto.shared.DelegateTo* annotation may help. DTO interface bellow contains getters, setters and with methods and one more complex method for getting full name of user.

```
@DTO
public interface User {
     String getFirstName();
     void setFirstName(String firstName);
     User withFirstName(String firstName);
     String getLastName();
     void setLastName(String lastName);
     User withLastName(String lastName);
     @DelegateTo(client = @DelegateRule(type = Util.class, method = "fullName"),
     server = @DelegateRule(type = Util.class, method = "fullName"))
     String getFullName();
}
```

For method getFullName add annotation DelegateTo. Annotations may contains different delegate rules for client and server code.

*DelegateTo* annotation

| Parameter | Description |
|-----------|-------------|
| client    | Rules for client code generator |
| server    | Rules for server code generator |

*DelegateRule* annotation

| Parameter | Description |
|-----------|-------------|
|type       | Class that contains method to delegate method call |
|method     | Name of method |

```
public class Util {
    public static String fullName(User user) {
        return user.getFirstName() + " " + user.getLastName();
    }
}
```

Fragment of generated code for method *getFullName()*:

```
public String getFullName() {
    return Util.fullName(this);
}
```

Requirements for methods to delegate DTO methods calls:
1. Method must be public and static.
2. Method must accept DTO interface as first parameter, if DTO method contains other parameters then the delegate method must accept the whole set of DTO method parameters starting from the second position.

For example:
```
@DelegateTo(client = @DelegateRule(type = Util.class, method = "fullName"),
    server = @DelegateRule(type = Util.class, method = "fullName"))
String getFullNameWithPrefix(String prefix);
```
Delegate method:

```
public static String fullName(User user, String prefix) {
    return prefix + " " + user.getFirstName() + " " + user.getLastName();
}
```
