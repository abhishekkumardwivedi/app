# Flatpak

## Install flatpak repo

* Add remote repo and check list of remote reposatories with details
`
sudo flatpak remote-add --from gnome https://sdk.gnome.org/gnome.flatpakrepo
flatpak remote-list -d
`
* Check content of remote repositories. Content could be list of app, runtime and SDK with version. We have added gnome so lets check gnome list
`
flatpak remote-ls -d gnome
`
* Let's install content from the above list. As we will be building app, we will need SDK and Runtime installed which is pre-requisit to build app.
`
sudo flatpak install gnome runtime/org.gnome.Platform//3.24
sudo flatpak install gnome runtime/org.gnome.Sdk//3.24
`
* We can check the list of installed Apps/SDKs/Runtimes
`
flatpak list
`
## Build application

Building application requires SDK and Runtime in place. We have already installed these in above steps. Now it time to build applcation. We can write applcation seperately crosscompile for the target platform and can wrap with flatpak app build system. And then finally we can put it in repo and can put in web server to make it available for installation. Alternatively we can build bundle file which could be directly installed on the system.

### Build an applcation
I have written java application here. One can write applcation in any language supprted by used SDK and Runtime, such as C, C++, python, nodejs, etc.
* Write application
`
vim hello.java

public class hello {

    public static void main(String[] args) {
        // Prints "Hello, World" to the terminal window.
        System.out.println("========== Hello World from JAVA ==============");
    }

}
`
* Compile application
`
javac hello.java
`
* Test application
`
java hello

#below message will be printed:

========== Hello World from JAVA ==============
`
By now we have executable application ready to be wrapped in Flatpak application.

### Build a flatpak application
To build an application for flatpak, we need to initialize the build directory and later keep building with all the needed components and finally finish the build. Flatpak application has directory structure. We can create this directory structure handwritten. Flatpak also provides some utilities, flatpak build, for this purpose and we will using this utlity.

To begin with flatpak applcation directory initialization, we need following details beforehand.
* Flatpak application name. This is the ID for application which is used for application to run and also seen in all the content lists when put into server repo.
* Flatpak driectory name
* Flatpak SDK to build the applcation.
* Flatpak Runtime for the applcation, corresponding to SDK.

We will use below command to initialize the build directory. Syntax could be found in "man flatpak build-init". Which will give much better details with multiple opetions and better it should be checked.
`
flatpak build-init appdir com.mysample.App org.gnome.Sdk/x86_64/3.24 org.gnome.Platform/x86_64/3.24
`
We have provided machine and version information for SDK and Runtime. Now lets see the directory structure.
`
tree

.
└── appdir
    ├── files
    ├── metadata
    └── var
        ├── run -> /run
        └── tmp
`
Now let's see the metadata.
`
cat appdir/metadata

[Application]
name=com.testapp.App
runtime=org.gnome.Platform/x86_64/3.24
sdk=org.gnome.Sdk/x86_64/3.24
`
We had built a java applcation earlier. Let's put it in flatpak project. For that we need to create app directory
`
flatpak build appdir mkdir /app/
`
See what are changes in directory structure
`
tree
.
├── appdir
│   ├── files
│   │   └── bin
│   ├── metadata
│   └── var
│       ├── run -> /run
│       └── tmp
`
files/bin directory is created. But there is no change in metadata.

* Install hello.class application to flatpak build.
`
flatpak build appdir install --mode=750 hello.class /app/bin
`

Let's see tree now
`
tree
├── appdir
│   ├── files
│   │   └── bin
│   │       └── hello.class
│   ├── metadata
│   └── var
│       ├── run -> /run
│       └── tmp
`
So, we placed hello.class inside bin directory. But still metadata is not updated. We have to add command in metadata for flatpak applcation to recognize what to execute when we run flatpak applcation. We will do it at build finish.
`
flatpak build-finish --command='java hello' appdir
`
Now our app is ready and can be executed. But before that let's review the metadata
`
cat appdir/metadata

[Application]
name=com.testapp.App
runtime=org.gnome.Platform/x86_64/3.24
sdk=org.gnome.Sdk/x86_64/3.24
command=java hello
`

## Export applcation to repo
`
flatpak build-export repo appdir stable
`
Here stable is optional. If don't provide any it will be exported as master. By this we have completed the application development and is ready to be installed and tested.

## Let's install application and test
Installing locally built application is same as remote application. We need to add remote and then install. And finally execute applcation and test the output.

`
sudo flatpak --user remote-add --no-gpg-verify local-repo repo
sudo flatpak --user install local-repo com.testapp.App
flatpak run com.testapp.App
`

# Reference
https://blogs.gnome.org/alexl/
