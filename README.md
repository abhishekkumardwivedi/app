# Flatpak

`
#List of remote reposatories added
flatpak remote-list -d

#Add remote repo
sudo flatpak remote-add --from gnome https://sdk.gnome.org/gnome.flatpakrepo

# Check added remote repo list with details
sudo flatpak remote-list -d

#check content of added repo, e.g. gnome (we just added)
flatpak remote-ls -d gnome

#Install runtime/sdk/app. Chose it from above remote content:
sudo flatpak install gnome runtime/org.gnome.Platform//3.24
sudo flatpak install gnome runtime/org.gnome.Sdk//3.24

#Check list of installed Apps/SDKs/Runtimes
flatpak list


####### Now lets build our own application and test ########

mkdir sample
cd sample

# lets initialize directory for build. It create the flatpak app directory structure and metadata
# what we need before hand: app name, directory for app, SDK to buid upon, Runtime corresponding to SDK.


flatpak build-init appdir com.testapp.App org.gnome.Sdk/x86_64/3.24 org.gnome.Platform/x86_64/3.24

# Now, let's see what we have created

tree

.
└── appdir
    ├── files
    ├── metadata
    └── var
        ├── run -> /run
        └── tmp


#metadata is the place to see the details

cat appdir/metadata


[Application]
name=com.testapp.App
runtime=org.gnome.Platform/x86_64/3.24
sdk=org.gnome.Sdk/x86_64/3.24

# So we have created a initial build directory by specifying app name, runtime and Sdk.

# Now lets add our sample applcation, writeen in any supported language and compiled.

# Lets create java main class and compile. Which will be put in flatpak package.

# vim hello.java

public class hello {

    public static void main(String[] args) {
        // Prints "Hello, World" to the terminal window.
        System.out.println("========== Hello World from JAVA ==============");
    }

}

# Let's compile java app and verify before putting into package
javac hello.java

# Test this app at first before putting to package
java hello
#below message will be printed:

========== Hello World from JAVA ==============


# Let's put this java app in flatpak project. Mind that java app is platform independent, so this app app will run on any device architecutre, installed with java.

flatpak build appdir mkdir /app/

# Now check tree command for what is added
tree
.
├── appdir
│   ├── files
│   │   └── bin
│   ├── metadata
│   └── var
│       ├── run -> /run
│       └── tmp

# what is update in metadata?
cat appdir/metadata
[Application]
name=com.testapp.App
runtime=org.gnome.Platform/x86_64/3.24
sdk=org.gnome.Sdk/x86_64/3.24

# So there is no update in metadata here but an app direcotry, /bin is created, which is shown in directory structure.

#Now we are ready to install app, lets install hello.class java app for flatpak build
flatpak build appdir install --mode=750 hello.class /app/bin

# Now check the tree, where we have our app, hello.class, copied
tree
├── appdir
│   ├── files
│   │   └── bin
│   │       └── hello.class
│   ├── metadata
│   └── var
│       ├── run -> /run
│       └── tmp

# But still metadata is not updated. This is because we have to exclusively add it in next command so that run command can find pur app and execute.

flatpak build-finish --command='java hello' appdir

# Now our app is ready and can be executed. But before that let's review the metadata
cat appdir/metadata

[Application]
name=com.testapp.App
runtime=org.gnome.Platform/x86_64/3.24
sdk=org.gnome.Sdk/x86_64/3.24
command=java hello

# Let's export this to repo
flatpak build-export repo appdir stable
sudo flatpak remote-add --no-gpg-verify example


`
# Reference
https://blogs.gnome.org/alexl/
