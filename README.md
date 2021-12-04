# ossec21
Suite à la génération du premier rapport, deux niveaux de vulnérabilité sont présentes (high et medium). Dans le but d'être le plus optimisé possible, la correction portera uniquement sur le niveau "high".

- Ensure gpgcheck Enabled for Local Packages
- Ensure Software Patches Installed
## Recherche du profil
```bash
data_stream="/usr/share/xml/scap/ssg/content/ssg-rhel7-ds-1.2.xml"
oscap info --fetch-remote-resources $data_stream
```
## Paramétrer les opérations
```bash
target="srv2"
type="$target-bp28m-before"
data_stream="/usr/share/xml/scap/ssg/content/ssg-rhel7-ds-1.2.xml"
profile="xccdf_org.ssgproject.content_profile_anssi_nt28_minimal"
cpe_dict="/usr/share/xml/scap/ssg/content/ssg-rhel7-cpe-dictionary.xml"
```
## Setup de la cible (installation du scanner)
```bash
ssh-keygen -b 4096 -t rsa -f $HOME/.ssh/id_rsa -q -N ""
target="srv2"
ssh-copy-id $target

ssh $target "yum -y install scap-security-guide"
```
## Setup de la station de contrôle
```bash
curl -L https://git.io/JMPci \
-o /usr/share/xml/scap/ssg/content/ssg-rhel7-ds-1.2.xml
type="$target-bp28m-before"
data_stream="/usr/share/xml/scap/ssg/content/ssg-rhel7-ds-1.2.xml"
cpe_dict="/usr/share/xml/scap/ssg/content/ssg-rhel7-cpe-dictionary.xml"
```
## Afficher des informations sur un Data Stream (DS) SCAP
```bash
data_stream="/usr/share/xml/scap/ssg/content/ssg-rhel7-ds-1.2.xml"
oscap info --fetch-remote-resources $data_stream
```
## Afficher des informations plus détaillées sur un profil
```bash
profile="xccdf_org.ssgproject.content_profile_anssi_nt28_minimal"
oscap info --profile $profile --fetch-remote-resources $data_stream
```
## Scan avec un Data Stream (DS) SCAP
```bash
type="$target-bp28m-before"
profile="xccdf_org.ssgproject.content_profile_anssi_nt28_minimal"
oscap-ssh --sudo root@$target 22 xccdf eval \
--fetch-remote-resource \
--profile $profile \
--results $type-results.xml \
--report $type-report.html \
--oval-results \
--cpe $cpe_dict \
$data_stream

ls -l srv2*
```
## Générer un guide de configuration
```bash
oscap xccdf generate guide \
--profile $profile \
--output $type-guide.html \
$type-results.xml
```
## Ouvir le pare-feu et lancer un service Web de fortune
```bash
firewall-cmd --add-port=8080/tcp
firewall-cmd --reload
python3 -m http.server 8080
```
## Scan d’un règle
```bash
rule1="xccdf_org.ssgproject.content_rule_ensure_gpgcheck_local_packages"
type="rule1-$target-bp28m-before"
profile="xccdf_org.ssgproject.content_profile_anssi_nt28_minimal"
oscap-ssh --sudo root@$target 22 xccdf eval \
--fetch-remote-resource \
--profile $profile \
--results rule1-$type-results.xml \
--report rule1-$type-report.html \
--oval-results \
--cpe $cpe_dict \
--rule $rule1 \
$data_stream
```
## Générer une remédiation Ansible
```bash
result_id=$(oscap info rule1-$type-results.xml | grep 'Result ID' | sed 's/[[:blank:]]Result ID: //')
oscap xccdf generate fix \
--fix-type ansible \
--output rule1-$type-playbook.yml \
--profile $profile \
--result-id $result_id \
rule1-$type-results.xml
```
## Vérifier un livre de jeu Ansible
```bash
ansible-playbook -i "$target," rule1-$type-playbook.yml --check --diff

ansible-playbook -i "$target," rule1-$type-playbook.yml
```
## Reboot de la machine
```bash
ansible all -i "$target," -m reboot
```
## Relancer l’audit
```bash
type="$target-bp28m-after"
profile="xccdf_org.ssgproject.content_profile_anssi_nt28_minimal"
oscap-ssh --sudo root@$target 22 xccdf eval \
--fetch-remote-resource \
--profile $profile \
--results $type-results.xml \
--report $type-report.html \
--oval-results \
--cpe $cpe_dict \
$data_stream
```
## Conclusion
Deux vulnérabilités de niveau "high" était présentes, une sur deux a pu être corrigée => Ensure gpgcheck Enabled for Local Packages.
