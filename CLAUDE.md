# CLAUDE.md

## Projekt

Rola Ansible: **install-keepalived-vip** — instaluje i konfiguruje keepalived do obslugi wirtualnych adresow IP (VIP) przez VRRP.

## Struktura

```
defaults/main.yml             # Domyslne wartosci `in_keepalived`
vars/main.yml                 # Stale wewnetrzne (sciezki, serwis, walidacja)
tasks/main.yml                # Glowne taski roli
templates/keepalived.conf.j2  # Szablon Jinja2 -> keepalived.conf
handlers/main.yml             # Restart keepalived (systemd + non-systemd)
meta/main.yml                 # Metadane Galaxy
examples/                     # Przyklady uzycia
```

## Konwencje nazewnictwa zmiennych

- `in_` — zmienne wejsciowe (publiczne), np. `in_keepalived`
- `var_` — zmienne wewnetrzne (vars/), np. `var_keepalived_config_path`
- `_` — fakty obliczane w runtime, np. `_keepalived_unicast_peers`

## Styl kodu

- Pelne kwalifikowane nazwy modulow: `ansible.builtin.package`, `ansible.builtin.template`
- Nazwy taskow w cudzyslowach: `- name: "Install keepalived"`
- `loop_control.loop_var` zamiast domyslnego `item`
- Warunki `when` jako lista (nawet dla jednego warunku)
- Logika obliczeniowa w taskach (`set_fact`), nie w szablonie Jinja2
- Handlery uzywaja wzorca `listen` z wariantami per service manager

## Jezyk

- Kod, nazwy taskow, zmienne, komunikaty bledow: **angielski**
- Komentarze w defaults/main.yml: **polski**
- Pliki .llm: **polski**

## Kluczowe mechanizmy

### Unicast peers (tasks/main.yml)

Priorytet rozwiazywania peerow:
1. `unicast.peers` — jawna lista IP (najwyzszy priorytet)
2. `unicast.peers_group` / `in_keepalived.unicast_peers_group` — hosty z grupy inventory
3. `play_hosts` — fallback na hosty w biezacym playu

Wynik: fakt `_keepalived_unicast_peers` (dict: nazwa instancji -> lista IP).
Szablon Jinja2 korzysta z tego faktu — nie duplikuje logiki.

### Walidacja konfiguracji

Gdy `in_keepalived.validate_config: true` (domyslnie), template uzywa `keepalived -t -f %s` do walidacji przed zapisem.

## Platformy

- Debian/Ubuntu, RHEL/Alma/Rocky (EL)
- Ansible >= 2.12

## Testy

Brak skonfigurowanych testow Molecule w repozytorium.
