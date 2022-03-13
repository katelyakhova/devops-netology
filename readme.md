2.4. Инструменты Git

1.Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.
$ git log -1 --format="%H %s" aefea
aefead2207ef7e2aa5dc81a34aedf0cad4c32545 Update CHANGELOG.md

2.Какому тегу соответствует коммит 85024d3?
$ git log -1 --format="%H %d" 85024d3
85024d3100126de36331c6982bfaac02cdab9e76  (tag: v0.12.23)

3.Сколько родителей у коммита b8d720? Напишите их хеши.
$ git log -1 --format="%P" b8d720
56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b

4.Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.
$ git log --format="%H %s" v0.12.23..v0.12.24
33ff1c03bb960b332be3af2e333462dde88b279e v0.12.24
b14b74c4939dcab573326f4e3ee2a62e23e12f89 [Website] vmc provider links
3f235065b9347a758efadc92295b540ee0a5e26e Update CHANGELOG.md
6ae64e247b332925b872447e9ce869657281c2bf registry: Fix panic when server is unreachable
5c619ca1baf2e21a155fcdb4c264cc9e24a2a353 website: Remove links to the getting started guide's old location
06275647e2b53d97d4f0a19a0fec11f6d69820b5 Update CHANGELOG.md
d5f9411f5108260320064349b757f55c09bc4b80 command: Fix bug when using terraform login on Windows
4b6d06cc5dcb78af637bbb19c198faff37a066ed Update CHANGELOG.md
dd01a35078f040ca984cdd349f18d0b67e486c35 Update CHANGELOG.md
225466bc3e5f35baa5d07197bbc079345b77525e Cleanup after v0.12.23 release

5.Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).
$ git grep --heading -e 'func providerSource('
provider_source.go
func providerSource(configs []*cliconfig.ProviderInstallation, services *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) {
$ git log --pretty=oneline -L :providerSource:provider_source.go 
5af1e6234ab6da412fb8637393c5a17a1b293663 main: Honor explicit provider_installation CLI config when present

diff --git a/provider_source.go b/provider_source.go
--- a/provider_source.go
+++ b/provider_source.go
@@ -20,6 +23,15 @@
-func providerSource(services *disco.Disco) getproviders.Source {
-       // We're not yet using the CLI config here because we've not implemented
-       // yet the new configuration constructs to customize provider search
-       // locations. That'll come later. For now, we just always use the
-       // implicit default provider source.
-       return implicitProviderSource(services)
+func providerSource(configs []*cliconfig.ProviderInstallation, services *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) {
+       if len(configs) == 0 {
+               // If there's no explicit installation configuration then we'll build
+               // up an implicit one with direct registry installation along with
+               // some automatically-selected local filesystem mirrors.
+               return implicitProviderSource(services), nil
+       }
+
+       // There should only be zero or one configurations, which is checked by
+       // the validation logic in the cliconfig package. Therefore we'll just
+       // ignore any additional configurations in here.
+       config := configs[0]
+       return explicitProviderSource(config, services)
+}
+
92d6a30bb4e8fbad0968a9915c6d90435a4a08f6 main: skip direct provider installation for providers available locally

diff --git a/provider_source.go b/provider_source.go
--- a/provider_source.go
+++ b/provider_source.go
@@ -19,5 +20,6 @@
 func providerSource(services *disco.Disco) getproviders.Source {
        // We're not yet using the CLI config here because we've not implemented
        // yet the new configuration constructs to customize provider search
-       // locations. That'll come later.
-       // For now, we have a fixed set of search directories:
+       // locations. That'll come later. For now, we just always use the
+       // implicit default provider source.
+       return implicitProviderSource(services)
8c928e83589d90a031f811fae52a81be7153e82f main: Consult local directories as potential mirrors of providers

diff --git a/provider_source.go b/provider_source.go
--- /dev/null
+++ b/provider_source.go
@@ -0,0 +19,5 @@
+func providerSource(services *disco.Disco) getproviders.Source {
+       // We're not yet using the CLI config here because we've not implemented
+       // yet the new configuration constructs to customize provider search
+       // locations. That'll come later.
+       // For now, we have a fixed set of search directories:

6.Найдите все коммиты в которых была изменена функция globalPluginDirs.
$ git grep --heading -e 'func globalPluginDirs'
plugins.go
func globalPluginDirs() []string {
$ git log --pretty=oneline -L :globalPluginDirs:plugins.go
78b12205587fe839f10d946ea3fdc06719decb05 Remove config.go and update things using its aliases

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,14 +18,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
-       dir, err := ConfigDir()
+       dir, err := cliconfig.ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
                ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }
 
        return ret
 }
52dbf94834cb970b510f2fba853a5b49ad9b1a46 keep .terraform.d/plugins for discovery

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,13 +16,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }
 
        return ret
 }
41ab0aef7a0fe030e84018973a64135b11abcd70 Add missing OS_ARCH dir to global plugin paths

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -14,12 +16,13 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
-               ret = append(ret, filepath.Join(dir, "plugins"))
+               machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }
 
        return ret
 }
66ebff90cdfaa6938f26f908c7ebad8d547fea17 move some more plugin search path logic to command

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,22 +14,12 @@
 func globalPluginDirs() []string {
        var ret []string
-
-       // Look in the same directory as the Terraform executable.
-       // If found, this replaces what we found in the config path.
-       exePath, err := osext.Executable()
-       if err != nil {
-               log.Printf("[ERROR] Error discovering exe directory: %s", err)
-       } else {
-               ret = append(ret, filepath.Dir(exePath))
-       }
-
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                ret = append(ret, filepath.Join(dir, "plugins"))
        }
 
        return ret
 }
8364383c359a6b738a436d1b7745ccdce178df47 Push plugin discovery down into command package

diff --git a/plugins.go b/plugins.go
--- /dev/null
+++ b/plugins.go
@@ -0,0 +16,22 @@
+func globalPluginDirs() []string {
+       var ret []string
+
+       // Look in the same directory as the Terraform executable.
+       // If found, this replaces what we found in the config path.
+       exePath, err := osext.Executable()
+       if err != nil {
+               log.Printf("[ERROR] Error discovering exe directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Dir(exePath))
+       }
+
+       // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
+       dir, err := ConfigDir()
+       if err != nil {
+               log.Printf("[ERROR] Error finding global config directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Join(dir, "plugins"))
+       }
+
+       return ret
+}
(END)

7.Кто автор функции synchronizedWriters?
$ git log -S"func synchronizedWriters" --pretty=format:'%h %an %ad %s'
bdfea50cc James Bardin Mon Nov 30 18:02:04 2020 -0500 remove unused
5ac311e2a Martin Atkins Wed May 3 16:25:41 2017 -0700 main: synchronize writes to VT100-faker on Windows
Автор в данном случае очевидно Martin Atkins, а James Bardin наоборот ее удалил
