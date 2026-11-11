# Lab Architecture

## 1. Overview

Purple Team Lab - изолированная виртуальная инфраструктура для моделирования кибератак, обнаружения угроз и проверки механизмов защиты.

Виртуальная среда развёрнута на базе KVM/QEMU с использованием libvirt.

Основные задачи инфраструктуры:

- Эмуляция корпоративной сети.
    
- Централизованное управление доменными компьютерами.
    
- Сбор событий безопасности с конечных узлов.
    
- Обнаружение подозрительной активности.
    
- Анализ инцидентов и проверка эффективности защитных мер.
    

## 2. Virtualization Environment

|Компонент|Конфигурация|
|---|---|
|Hypervisor|KVM / QEMU|
|Virtual Machine Manager|libvirt|
|Host OS|Arch Linux|
|Virtual Networks|libvirt virtual networks|
|Network Segmentation|LAN / SERVER / DMZ / ATTACK|

## 3. Network Architecture

Лабораторная инфраструктура разделена на четыре виртуальные сети.

|Network|Subnet|Gateway|Purpose|
|---|---|---|---|
|LAN|10.10.10.0/24|10.10.10.1|Active Directory и Windows-клиенты|
|SERVER|10.10.20.0/24|10.10.20.1|SIEM и серверная инфраструктура|
|DMZ|10.10.30.0/24|10.10.30.1|Тестовые сервисы|
|ATTACK|10.10.40.0/24|10.10.40.1|Среда моделирования атак|

Маршрутизация между сегментами осуществляется через OPNsense.

Сегментация позволяет разделять инфраструктурные компоненты и контролировать сетевое взаимодействие между ними.

## 4. Virtual Machines

### FW01 - OPNsense

**Назначение:** межсетевой экран и маршрутизатор лабораторной сети.

Функции:

- Маршрутизация между виртуальными сетями.
    
- Контроль сетевого взаимодействия.
    
- Сегментация инфраструктуры.
    
- Применение правил межсетевого экранирования.
    

Интерфейсы:

- LAN: `10.10.10.1`
    
- SERVER: `10.10.20.1`
    
- DMZ: `10.10.30.1`
    
- ATTACK: `10.10.40.1`
    

### DC01 - Domain Controller

**Операционная система:** Windows Server 2022.

**IP-адрес:** `10.10.10.10`

**Домен:** `lab.local`

**NetBIOS:** `LAB`

Развёрнутые службы:

- Active Directory Domain Services.
    
- DNS Server.
    
- Group Policy.
    

Структура Active Directory включает организационные подразделения для пользователей, компьютеров, серверов, сервисных и административных учётных записей.

На сервере установлен Wazuh Agent и настроен сбор событий Windows и Sysmon.

### WAZUH01 - SIEM Server

**Операционная система:** Ubuntu Server 24.04.5.

**IP-адрес:** `10.10.20.10`

**Hostname:** `wazuh01`

Развёрнут Wazuh All-in-One:

- Wazuh Manager.
    
- Wazuh Indexer.
    
- Wazuh Dashboard.
    

Основные функции:

- Приём событий от агентов.
    
- Анализ и корреляция событий.
    
- Применение штатных и пользовательских правил.
    
- Хранение и поиск событий.
    
- Визуализация обнаружений в Dashboard.
    

Веб-интерфейс доступен по адресу:

`https://10.10.20.10`

### Windows 11 - Domain Client

**Назначение:** рабочая станция доменного пользователя.

Функции:

- Работа в домене `lab.local`.
    
- Получение доменных политик.
    
- Взаимодействие с Domain Controller.
    
- Использование в качестве дополнительной конечной точки для тестирования.
    

### Ubuntu Server - DMZ

**Назначение:** размещение тестовых сервисов в отдельном сетевом сегменте.

Планируемое использование:

- Развёртывание DVWA.
    
- Моделирование атак на веб-приложения.
    
- Анализ сетевых событий и активности приложений.
    

### Kali Linux - Attack Machine

**Назначение:** специализированная машина для моделирования атак.

Планируемое использование:

- Сканирование инфраструктуры.
    
- Тестирование сетевых сервисов.
    
- Моделирование атак на Active Directory.
    
- Проверка обнаружения атак средствами Wazuh.
    

## 5. Monitoring Architecture

Основной источник телеметрии Windows - Microsoft Sysmon.

На `DC01` настроен сбор следующих журналов:

|Log Source|Назначение|
|---|---|
|Sysmon/Operational|События процессов, файловой системы, сети и другие системные события|
|PowerShell/Operational|События PowerShell|
|Security|События безопасности Windows|

Wazuh Agent передаёт собранные события на Wazuh Manager (`10.10.20.10`).

Wazuh Manager анализирует поступающие данные с использованием правил обнаружения.

## 6. Detection Workflow

```
       Attack Simulation
              |
              v
        Target System
              |
              v
       Sysmon / Windows
         Event Logs
              |
              v
         Wazuh Agent
              |
              v
        Wazuh Manager
              |
              v
       Detection Engine
              |
              v
        Wazuh Indexer
              |
              v
        Wazuh Dashboard
              |
              v
       Investigation
              |
              v
          Hardening
              |
              v
            Retest
```

## 7. Security Objectives

Лаборатория предназначена для практического изучения следующих направлений:

- SIEM и централизованный мониторинг.
    
- Endpoint Detection and Response (EDR).
    
- Windows Event Logging.
    
- Sysmon Telemetry.
    
- Active Directory Security.
    
- Detection Engineering.
    
- MITRE ATT&CK.
    
- Incident Investigation.
    
- Infrastructure Hardening.
    

## 8. Current Implementation Status

|Component|Status|
|---|---|
|KVM / libvirt|Deployed|
|OPNsense|Configured|
|Network Segmentation|Configured|
|Active Directory|Deployed|
|Windows Domain Client|Joined to Domain|
|Wazuh All-in-One|Deployed|
|Wazuh Agent on DC01|Active|
|Sysmon|Installed and Configured|
|Custom Detection Rule|Tested|
|DMZ Attack Scenarios|In Progress|
|Hardening and Retest|Planned|

## 9. Project Workflow

Проект развивается по следующему циклу:

**Attack → Detect → Investigate → Harden → Retest**

Каждый сценарий предполагает контролируемое моделирование активности, регистрацию событий, проверку обнаружения и документирование результатов.