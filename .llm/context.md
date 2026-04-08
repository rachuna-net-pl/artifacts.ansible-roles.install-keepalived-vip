# install-keepalived-vip

Rola Ansible instalująca i konfigurująca **keepalived** do obsługi wirtualnych adresów IP (VIP) za pomocą protokołu VRRP.

## Struktura projektu

```
defaults/main.yml          # Domyślne wartości zmiennej `in_keepalived`
vars/main.yml              # Stałe wewnętrzne (ścieżki, nazwa serwisu, komenda walidacji)
tasks/main.yml             # Główne taski roli
templates/keepalived.conf.j2  # Szablon Jinja2 generujący keepalived.conf
handlers/main.yml          # Handlery (restart keepalived — systemd i non-systemd)
meta/main.yml              # Metadane Galaxy (Debian, EL, Ansible >= 2.12)
```

## Przepływ tasków

1. Instalacja pakietu `keepalived` (agnostycznie wobec menedżera pakietów)
2. Utworzenie katalogu konfiguracji `/etc/keepalived/`
3. Obliczenie unicast peers (`set_fact` → `_keepalived_unicast_peers`) — wynik współdzielony między assert a szablonem
4. Walidacja — assert sprawdza, że unicast-enabled instancje mają zdefiniowanych peerów
5. Renderowanie `keepalived.conf` z opcjonalną walidacją (`keepalived -t -f`)
6. Uruchomienie serwisu (systemd) z graceful skip na non-systemd

## Główna zmienna wejściowa: `in_keepalived`

| Klucz | Typ | Opis |
|---|---|---|
| `validate_config` | bool (true) | Walidacja konfiguracji przed zapisem |
| `global_defs.router_id` | string | Identyfikator routera (default: inventory_hostname) |
| `vrrp_scripts` | list | Definicje skryptów health-check (name, script, interval, weight, fall, rise) |
| `unicast_peers_group` | string | Globalna grupa inventory do auto-discovery peerów |
| `instances` | list | Lista instancji VRRP |

### Pola instancji VRRP

**Wymagane:** `name`, `interface`, `virtual_router_id`, `priority`, `virtual_ipaddresses`

**Opcjonalne:**
- `state` (default: BACKUP), `advert_int` (default: 1)
- `nopreempt` (bool) — wyłącza preempcję
- `unicast` — obiekt: `enabled`, `peers`, `peers_group`, `src_ip`
- `authentication` — obiekt: `auth_type` (default: PASS), `auth_pass`
- `track_interface` — lista interfejsów do monitorowania
- `track_script` — lista nazw skryptów VRRP do śledzenia
- `extra_lines` — surowe linie wstawiane do bloku instancji

## Zmienne wewnętrzne (vars/main.yml)

- `var_keepalived_service_name`: keepalived
- `var_keepalived_config_path`: /etc/keepalived/keepalived.conf
- `var_keepalived_bin`: /usr/sbin/keepalived
- `var_keepalived_validate_cmd`: keepalived -t -f %s

## Mechanizm unicast peers

Logika (w `tasks/main.yml`, wynik w `_keepalived_unicast_peers`):
1. Jeśli `unicast.peers` podane ręcznie → użyj ich
2. W przeciwnym razie weź hosty z grupy `unicast.peers_group` (lub globalnego `in_keepalived.unicast_peers_group`)
3. Jeśli brak grupy → użyj `play_hosts`
4. Odfiltruj bieżący host, wyciągnij `ansible_default_ipv4.address` z hostvars

Szablon Jinja2 korzysta z obliczonego faktu `_keepalived_unicast_peers[inst.name]` — nie duplikuje logiki.
