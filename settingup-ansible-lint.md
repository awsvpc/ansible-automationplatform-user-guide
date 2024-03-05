<pre>
  For beginners, working with YAML files can sometimes be challenging. Mistakes such as incorrect indentation or using a list instead of a dictionary can result in errors, leading to a frustrating experience with this powerful language.

To address these challenges, I attempted to install a YAML linter extension in Visual Studio Code (VSCode). Additionally, as I started working with Ansible, I received advice to use the Ansible extension for VSCode. However, after installing the extension, I encountered an error message that appeared in the VSCode error window: “Unable to parse YAML playbooks — ERROR: ‘Ansible-lint is not available. Kindly check the path or disable validation using ansible-lint’”.

Resolving this error can be challenging, especially if you are unfamiliar with Ansible lint and how to add its installation folder to the PATH environment variable. To help you navigate this process, here are some steps you can follow:

Firstly, ensure that you have Python’s pip package manager installed. If it is not installed, you can install it by following the appropriate instructions for your operating system.

// Classic update && upgrade of apt

sudo apt update
sudo apt upgrade

// Installation of python pip

sudo apt install python3-pip

// Below should return pip help

pip3
Once pip is installed, open a terminal or command prompt and execute the following command to install Ansible lint:


// Installing...

pip3 install ansible-lint

// Check installation success

ansible-lint --version

Normally, you should encounter the “unknown command” error, which occurs because you need to add the installation folder to your PATH. In my case, I am using the zsh shell, so I will edit the .zshrc file to include the following line:

// .zshrc

export PATH="$HOME/.local/bin:$PATH"

// reload zshrc configuration

sources ~/.zshrc
You can now try the ansible-lint --version command, and it should return the following result:

ansible-lint 6.14.6 using ansible 2.12.4

If everything is set up correctly, you should see your YAML file highlighted with helpful colors and receive error notifications regarding indentation or incorrect commands.

I hope this information proves helpful to you!


</pre>
