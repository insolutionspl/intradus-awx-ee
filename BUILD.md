# Budowanie customowego obrazu

## Instalacja `ansible-builder`

Jeśli mamy Pythona 3.9, to super - wykonujemy polecenia do `pip` w sposób jak niżej:

```bash
pip install ansible-builder==3.0.0
```

Jeśli nie, to musimy zainstalować Pythona 3.9 - dla Ubuntu wygląda to tak:

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.9
```

i komendy do `pip` wykonywać tak:

```bash
python3.9 -m pip install ansible-builder==3.0.0
```

Dokładnie zresztą taką komendę należy uruchomić, żeby zainstalować `ansible-builder`. Oczywiście uruchamiamy ją wewnątrz katalogu z tym repozytorium.

Jeśli wersja Pythona nie będzie pasować, dostaniemy komunikat podobny do poniższego:

```bash
ERROR: Package 'ansible-builder' requires a different Python: 3.8.10 not in '>=3.9'
```

Po instalacji konieczne jest sourcowanie środowiska, żeby `ansible-builder` było widoczne w ścieżce:

```bash
source ~/.profile
```

w innym wypadku dostaniemy komunikat jak niżej przy próbie użycia `ansible-builder`:

```bash
ansible-builder: command not found
```

A czemu w ogóle wersja 3.0.0 skryptu `ansible-builder`? Ponieważ obecna wersja repozytorium `awx-ee`, które sforkowaliśmy, używa kluczy YAML w pliku `execution-environment.yml`, które są dostępne dopiero od 3.x. Jeśli użyjemy wersji 2.x, to dostaniemy komunikat:

```bash
ansible_builder.exceptions.DefinitionError:
Error: Unknown yaml key(s), {'images'}, found in the definition file.

Allowed options are:
{'dependencies', 'ansible_config', 'version', 'additional_build_steps', 'build_arg_defaults'}
```

## Budowanie obrazu

W katalogu `awx-ee` uruchamiamy:

```bash
ansible-builder build -v3 -t intradus/intradus-awx-custom-ee --container-runtime=docker 
```

Jak widać w komendzie, podajemy nazwę obrazu, który chcemy zbudować. W tym przypadku jest to `intradus/intradus-awx-custom-ee`. Do tego wskazujemy runtime kontenera, którym jest `docker`, ponieważ domyślnie `ansible-builder` używa `podman`a.

Po zbudowaniu obrazu, możemy go wrzucić do naszego repozytorium na DockerHubie:

```bash
docker push intradus/intradus-awx-custom-ee
```

Oczywiście musimy być zautoryzowani.

## Lokalna weryfikacja zbudowanego obrazu

Po co w ogóle budujemy customowy obraz? Np. po to, żeby doinstalować pakiety, które są potrzebne do uruchomienia naszych playbooków. W tym przypadku jest to `hvac` - biblioteka do Vaulta. W pliku `execution-environment.yml` dodaliśmy w sekcji `dependencies` pozycję `hvac`. Po zbudowaniu obrazu, możemy go uruchomić lokalnie, zanim jeszcze wrzucimy go na DockerHuba, żeby zwerfikować, czy wszystko działa jak należy i np. pakiet `hvac` jest zainstalowany.

```bash
docker run --user 0 --entrypoint /bin/bash -it --rm intradus/intradus-awx-custom-ee:latest
```

i wewnątrz kontenera:

```bash
pip show hvac
```

To samo możemy zresztą już zrobić w środowisku Kubernetes, w którym użyty jest nasz nowy obraz - jeśli tak, to pod `awx-instance-task-XXX` powinien wykorzystywać już nowy obraz, który zbudowaliśmy. Wchodzimy do niego i sprawdzamy, czy pakiet `hvac` jest zainstalowany:

```bash
kubectl exec -it -n awx awx-instance-task-5488597bc4-znpz5 -c awx-instance-ee bash
```

i wewnątrz kontenera:

```bash
pip show hvac
```

