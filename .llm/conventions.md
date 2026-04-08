# Konwencje projektu

## Język

- Nazwy tasków, zmiennych i komentarzy w kodzie: **angielski**
- Komunikaty błędów (fail_msg): **angielski**
- Komentarze w defaults/main.yml: **polski** (dokumentacja dla użytkownika)
- Pliki .llm: **polski**

## Nazewnictwo zmiennych

- Zmienne wejściowe (publiczne): prefix `in_` → `in_keepalived`
- Zmienne wewnętrzne (vars/): prefix `var_` → `var_keepalived_config_path`
- Fakty obliczane w runtime: prefix `_` → `_keepalived_unicast_peers`

## Styl tasków

- Pełne kwalifikowane nazwy modułów: `ansible.builtin.package`, `ansible.builtin.template`, etc.
- Nazwy tasków w cudzysłowach: `- name: "Install keepalived"`
- `loop_control.loop_var` zamiast domyślnego `item` w pętlach
- Warunki `when` jako lista (nawet dla jednego warunku)

## Szablon Jinja2

- Domyślne wartości przez filtr `| default()` — nigdy nie zakładaj, że zmienna istnieje
- Logika obliczeniowa (np. unicast peers) powinna być w taskach (`set_fact`), nie duplikowana w szablonie
- Szablon odwołuje się do obliczonych faktów (np. `_keepalived_unicast_peers[inst.name]`)

## Handlery

- Wzorzec `listen` z wariantami per service manager (systemd / non-systemd)
- Nazwa nasłuchu: `"Restart keepalived"`

## Platformy docelowe

- Debian (all versions)
- EL / RHEL (all versions)
- Ansible >= 2.12
