# Purple Team Lab

**Практическая лаборатория информационной безопасности для изучения атак, обнаружения угроз и укрепления инфраструктуры.**

Проект демонстрирует полный цикл Purple Team: моделирование атак на изолированной инфраструктуре, сбор событий, обнаружение подозрительной активности средствами SIEM, анализ результатов и последующее усиление защиты.

## Цели проекта

- Развернуть изолированную инфраструктуру на базе виртуальных машин.
    
- Создать доменную среду Active Directory.
    
- Настроить централизованный сбор и анализ событий с помощью Wazuh.
    
- Организовать сбор системной телеметрии Windows с помощью Sysmon.
    
- Разрабатывать и тестировать собственные правила обнаружения.
    
- Проводить контролируемые симуляции атак.
    
- Анализировать обнаруженные события и сопоставлять их с MITRE ATT&CK.
    
- Практиковать Hardening и повторное тестирование инфраструктуры.
    

## Архитектура лаборатории

### Основные компоненты

|Виртуальная машина|Назначение|IP-адрес|
|---|---|---|
|FW01|OPNsense, маршрутизация и сегментация сети|10.10.x.1|
|DC01|Windows Server 2022, Active Directory, DNS|10.10.10.10|
|WAZUH01|Ubuntu Server 24.04, Wazuh All-in-One|10.10.20.10|
|Windows 11|Доменная клиентская машина|DHCP|
|Ubuntu Server|Сервер DMZ, тестовые сервисы|10.10.30.x|
|Kali Linux|Среда для моделирования атак|10.10.40.x|

### Сегментация сети

|Сеть|Подсеть|Назначение|
|---|---|---|
|LAN|10.10.10.0/24|Доменная инфраструктура|
|SERVER|10.10.20.0/24|Серверы мониторинга|
|DMZ|10.10.30.0/24|Тестовые сервисы|
|ATTACK|10.10.40.0/24|Среда моделирования атак|

## Используемые технологии

- **Virtualization:** KVM / QEMU, libvirt
    
- **Firewall:** OPNsense
    
- **Operating Systems:** Arch Linux, Ubuntu Server, Windows Server 2022, Windows 11, Kali Linux
    
- **Directory Services:** Active Directory, DNS, Group Policy
    
- **SIEM:** Wazuh Manager, Indexer, Dashboard
    
- **Endpoint Telemetry:** Microsoft Sysmon
    
- **Threat Intelligence:** MITRE ATT&CK
    
- **Scripting:** PowerShell, Bash
    

## Реализованные механизмы обнаружения

### PowerShell EncodedCommand

**Цель:** обнаружение запуска PowerShell с использованием закодированных команд.

**MITRE ATT&CK:** T1059.001 — PowerShell.

**Источник телеметрии:** Sysmon Event ID 1 — Process Create.

#### Собственное правило Wazuh

Файл: `/var/ossec/etc/rules/local_rules.xml`

```
<group name="local,sysmon,windows,">
  <rule id="100100" level="8">
    <if_sid>61603</if_sid>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)-enc(odedcommand)?\s+</field>
    <description>PowerShell executed with EncodedCommand parameter</description>
    <mitre>
      <id>T1059.001</id>
    </mitre>
  </rule>
</group>
```

**Результат тестирования:**

- Правило `100100` успешно обнаружило тестовый запуск PowerShell с параметром `-enc`.
    
- Получен алерт уровня 8.
    
- В событии подтверждены агент `DC01`, командная строка и MITRE ATT&CK T1059.001.
    
- Дополнительно сработало штатное правило Wazuh `92057` уровня 12.
    

**Статус:** успешно протестировано.

## Принцип работы системы обнаружения

```
Windows Server (DC01)
        |
        v
    PowerShell
        |
        v
       Sysmon
        |
        v
    Wazuh Agent
        |
        v
   Wazuh Manager
        |
        v
  Detection Rules
        |
        v
  Wazuh Dashboard
        |
        v
 Analysis & Investigation
```

## Структура проекта

```
purple-team-lab/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── network.md
│   └── setup.md
│
├── detections/
│   └── wazuh/
│       └── local_rules.xml
│
├── attacks/
│   └── powershell-encodedcommand/
│       ├── attack.md
│       └── screenshots/
│
├── reports/
│   └── 2026-11-11.md
│
└── screenshots/
    ├── infrastructure/
    └── wazuh/
```

## Дальнейшие этапы

- Реализация дополнительных сценариев атак.
    
- Разработка новых правил обнаружения.
    
- Исследование событий безопасности Windows и Active Directory.
    
- Hardening доменной инфраструктуры.
    
- Повторное тестирование после изменения конфигурации.
    
- Подготовка incident reports.
    
- Документирование результатов и оформление GitHub-портфолио.
    

## Статус проекта

Проект находится на этапе практической реализации обнаружения угроз.

Уже развёрнута инфраструктура мониторинга Wazuh, подключён Windows Server, настроен сбор телеметрии Sysmon и успешно протестировано собственное правило обнаружения PowerShell EncodedCommand.

**Основной подход: Attack → Detect → Investigate → Harden → Retest.**