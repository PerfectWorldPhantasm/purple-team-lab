# Wazuh Custom Detection Rules

## 1. Overview

В проекте используются пользовательские правила Wazuh для обнаружения подозрительной активности на Windows Server.

Правила размещаются на сервере `WAZUH01` в файле:

`/var/ossec/etc/rules/local_rules.xml`

После изменения конфигурации выполняется проверка синтаксиса и перезапуск Wazuh Manager.

## 2. Rule 100100 - PowerShell EncodedCommand

**Назначение:** обнаружение запуска PowerShell с использованием параметров `-enc` и `-EncodedCommand`.

**MITRE ATT&CK:** T1059.001 - PowerShell.

**Источник события:** Sysmon Event ID 1 - Process Create.

### XML Configuration

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

### Detection Logic

1. Правило наследует контекст базового правила Sysmon `61603`.
    
2. Анализируется поле `win.eventdata.commandLine`.
    
3. Регулярное выражение ищет параметры `-enc` и `-EncodedCommand`.
    
4. При совпадении создаётся алерт уровня 8.
    
5. Событие сопоставляется с техникой MITRE ATT&CK `T1059.001`.
    

### Test Scenario

Для проверки правила на `DC01` была выполнена контролируемая симуляция запуска PowerShell с закодированной командой.

```
$c = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('Write-Output "PURPLE_LAB_TEST"')); powershell.exe -NoProfile -EncodedCommand $c
```

### Detection Result

<badge color="success">Successfully Detected</badge>

В Wazuh Dashboard получен алерт со следующими параметрами:

|Поле|Значение|
|---|---|
|Agent|DC01|
|Agent ID|001|
|Rule ID|100100|
|Rule Level|8|
|Event ID|1|
|Description|PowerShell executed with EncodedCommand parameter|
|MITRE ATT&CK|T1059.001|

Срабатывание подтверждено полученным JSON-событием.

## 3. Built-in Rule 92057

Дополнительно при тестировании сработало штатное правило Wazuh `92057`.

|Поле|Значение|
|---|---|
|Rule ID|92057|
|Rule Level|12|
|Event ID|1|
|Description|Powershell.exe spawned a powershell process which executed a base64 encoded command|
|MITRE ATT&CK|T1059.001|

Это подтверждает, что штатный механизм обнаружения Wazuh также распознаёт исследуемую активность.

## 4. Testing Summary

|Rule|Detection|Result|
|---|---|---|
|100100|Custom PowerShell EncodedCommand Detection|Successful|
|92057|Built-in PowerShell EncodedCommand Detection|Successful|

## 5. Conclusion

В ходе тестирования подтверждена работоспособность собственного правила Wazuh `100100`.

Правило обнаруживает запуск PowerShell с закодированной командой, анализируя телеметрию Sysmon Event ID 1.

Дополнительно подтверждено срабатывание штатного правила `92057`.

Таким образом, лаборатория поддерживает как использование встроенных механизмов обнаружения Wazuh, так и разработку собственных правил Detection Engineering.