### Update all NVM global packages

1. Install nvm on wsl: https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl
2. Install any version you want with nvm install
3. nano $NVM_DIR/default_packages
4. fill in which pacakges you want to install globally, for instance:

```
yarn
tsc
```

5. mkdir $NVM_DIR/bin
6. cd !$
7. Create nvm_install_global_packages.sh and fill it with the version:

```
#!/bin/bash

# Path to the default-packages file
DEFAULT_PACKAGES_FILE="$NVM_DIR/default-packages"

# Source nvm script to make nvm command available
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm

# Ensure the default-packages file exists
if [ -f "$DEFAULT_PACKAGES_FILE" ]; then
  # Get the list of installed Node.js versions
  nvm ls --no-colors | while read -r line; do
    # Extract the version number
    if [[ "$line" =~ v([0-9]+\.[0-9]+\.[0-9]+) ]]; then
      version="${BASH_REMATCH[1]}"
      # Stop processing once we reach the default version
      if [[ "$line" == *"default"* ]]; then
        break
      fi
      echo "Removing all global packages for Node.js $version"
      nvm exec $version npm ls -g --depth=0 --parseable=true | grep -v npm | xargs npm uninstall -g
      echo "All global packages removed for Node.js $version"

      # Read each line (package name) from the default-packages file
      while IFS= read -r package || [ -n "$package" ]; do
        # Skip comments and blank lines
        if [[ "$package" =~ ^#.*$ || -z "$package" ]]; then
          continue
        fi
        echo "Installing $package for Node.js $version"
        nvm exec $version npm install -g $package
      done < "$DEFAULT_PACKAGES_FILE"
    fi
  done
else
  echo "default-packages file not found."
fi
```

8. > chmod +x nvm_install_global_packages.sh
9. Add to .bashrc:
   > export PATH=$PATH:$NVM_DIR/bin
10. source ~/.bashrc
11. use it: nvm_install_global_packages
