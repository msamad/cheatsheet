# cheatsheet

Things that I don't remember but use every now and then.

## AutoHotkey

To quickly switch between window/terminal/IDE etc

- Map window/terminal/IDE to shortcut key using `Right Alt + key` e.g. `Right Alt + 1` to map the active windows
- Switch to windows using `Right Ctrl + key` e.g. `Right Alt + 1`

Note: 
- Only on windows https://www.autohotkey.com/
- Does not work with multi tabs. e.g. chrome etc
- The script assumes it is at `C:\autohotkey\autohotkey.ahk`


```
; Win+Shift+Break - Edit this file
#+Delete::
	Run "C:\Program Files (x86)\Notepad++\notepad++.exe" "C:\autohotkey\autohotkey.ahk"
	Return
; Win+Shift+Enter - Run this file
#+Enter::
	Run "C:\autohotkey\autohotkey.ahk" 
	Return
RAlt & 1::
RAlt & 2::
RAlt & 3::
RAlt & 4::
RAlt & 5::
RAlt & 6::
RAlt & 7::
RAlt & 8::
RAlt & 9::
RAlt & 0::
RAlt & -::
RAlt & +::
RAlt & `::
	KeyPressed = %A_ThisHotkey%
	StringRight, KeyNumber, KeyPressed, 1
	;MsgBox, %KeyNumber% was pressed
	WinGet, KeyValue, PID, A
	;WinGetActiveTitle, KeyValue
	IniWrite, %KeyValue%, C:\autohotkey\autohotkey.ini, SHORTCUT, Key%KeyNumber%
	;MsgBox, Window mapped %KeyValue% to %KeyNumber%
	TrayTip, AutoHotKey, Window mapped %KeyValue% to %KeyNumber%, 1, 1
	Return
RCtrl & 1::
RCtrl & 2::
RCtrl & 3::
RCtrl & 4::
RCtrl & 5::
RCtrl & 6::
RCtrl & 7::
RCtrl & 8::
RCtrl & 9::
RCtrl & 0::
RCtrl & -::
RCtrl & +::
RCtrl & `::
	KeyPressed = %A_ThisHotkey%
	StringRight, KeyNumber, KeyPressed, 1
	;MsgBox, %KeyNumber% was pressed
	IniRead, KeyValue, C:\autohotkey\autohotkey.ini, SHORTCUT, Key%KeyNumber%
	SetTitleMatchMode 3
	;Msgbox, Looking for window %KeyValue% to %KeyNumber%
	;IfWInExist, %KeyValue%
	;{
	;	WinActivate
	;}
	WinActivate, ahk_pid %KeyValue%
	Return
```

## openssl

Get certificates with chain from server
```
openssl s_client -showcerts -servername google.com -connect google.com:443 </dev/null
```

Output expiry date of the certificate
```
echo | openssl s_client -servername google.com -connect google.com:443 2>/dev/null | openssl x509 -noout -dates
```

## kubectl
Note: Working on zsh shell

Run command against each namespace except ones that include kube in their name

https://kubernetes.io/docs/tasks/access-application-cluster/list-all-running-container-images/

```
# use jsonpath
kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' |\
tr -s '[[:space:]]' '\n' |\
sort |\
grep -iv "kube" |\
xargs -I namespaceParameter kubectl get pods -n namespaceParameter

# use awk
kubectl get namespaces -A --no-headers=true | \
awk '!/kube/{print $1}' | \
xargs -I namespaceParameter kubectl get pods -n namespaceParameter


# build something using jsonpath
kubectl get pods -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{"."}{.metadata.namespace}' | sort
```

Get events sorted by time
```
kubectl get events --sort-by='.metadata.creationTimestamp'
```

Show a table of quota in all namespaces
```
kubectl get quota --all-namespaces -o custom-columns=\
NAMESPACE:.metadata.namespace,\
CPU_REQUEST:.spec.hard.requests\\.cpu,\
CPU_LIMIT:.spec.hard.limits\\.cpu,\
MEMORY_REQUEST:.spec.hard.requests\\.memory,\
MEMORY_LIMIT:.spec.hard.limits\\.memory
```

Show a table of limits in all namespaces 
```
# Double quotes are required here for custom-columns in zsh shell

kubectl get limits --all-namespaces -o custom-columns="\
NAMESPACE:.metadata.namespace,\
DEFAULT_CPU_REQUEST:.spec.limits[*].defaultRequest.cpu,\
DEFAULT_CPU_LIMIT:.spec.limits[*].default.cpu,\
MAX_CPU_ALLOWED:.spec.limits[*].max.cpu,\
DEFAULT_MEMORY_REQUEST:.spec.limits[*].defaultRequest.memory,\
DEFAULT_MEMORY_LIMIT:.spec.limits[*].default.memory,\
MAX_MEMORY_ALLOWED:.spec.limits[*].max.memory"
```

Export secrets and remove selfLink, timestamp etc

https://github.com/kubernetes/kubernetes/issues/24873
```
kubectl get secrets -o yaml | sed -e '/resourceVersion: "[0-9]\+"/d' -e '/uid: [a-z0-9-]\+/d' -e '/selfLink: [a-z0-9A-Z/]\+/d' -e '/status:/d' -e '/phase:/d' -e '/creationTimestamp:/d' > secrets.yaml
```

## kubctx & kubens

https://github.com/ahmetb/kubectx

A hack to get `kubens` working with multiple terminals. Create a copy of kube config file for each terminal

https://github.com/ahmetb/kubectx/issues/12

https://gist.github.com/stuart-warren/e5d22f51ee0affd17cbd459ddf2f67c9


Add this in ~/.zshrc file
```
# Create a single merged config file which has all the contexts
export KUBECONFIG=~/.kube/config
# Create a temp kubeconfig file per terminal to make sure kubectx and kubens changes are not global
# This allows to work with multiple terminal windows.
export KUBECTXTTYCONFIG="${HOME}/.kube/tty/$(basename $(tty) 2>/dev/null || echo 'notty')"
mkdir -p "$(dirname $KUBECTXTTYCONFIG)"
cp -f ${KUBECONFIG} ${KUBECTXTTYCONFIG}
export KUBECONFIG="${KUBECTXTTYCONFIG}"
```

# amazonlinux2

List all certs
```
cat /etc/pki/tls/certs/ca-bundle.crt | grep '#'
```

# gitbash

Avoid gitbash path problems
```
MSYS_NO_PATHCONV=1
```
