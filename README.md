# Dáta z Home assistant priamo na živý obraz 

Posielanie je spravené pomocou REST príkazy HTTP PUT. Po nastavení v HA sa budú hodnotu periodicky posielať na zivyobraz.eu, kde môžu byť použité na tvorbu e-paper obrazov. 

# Návod
1. Zisti zivyobraz kľúč `import_key`. Je zorazený v záložke "Hodnoty". Celý link vyzerá nejako takto: `http://in.zivyobraz.eu/?import_key=tvujklic`
2. Pridaj do configuration.yaml tieto položky a uprav `import_key` na svoj kľúč.
```yaml
input_text:
  zivy_obraz_hodnoty:
    #HTTP POST hodnoty pre Zivy Obraz. Spusti RESTful Command zivy_obraz_post na poslanie.
    #meno=hodnota&meno2=hodnota2
    name: Zivy Obraz hodnoty
    initial: not initalized
rest_command:
  zivy_obraz_post:
    url: "http://in.zivyobraz.eu/?import_key=TVUJ_KLIC&{{ states('input_text.zivy_obraz_hodnoty') }}"
```
3. Reštartuj home assistant
4. Vytvor automatizáciu s vlastnými hodnotami senzorov

    Príklad (náhľad ako to vyzerá v GUI je nižšie):
```yaml
alias: Zivy obraz update
description: "aktualizuj zivyobraz kazdu minutu"
trigger:
  - platform: time_pattern
    minutes: /1
condition: []
action:
  - service: input_text.set_value
    data:
      value: >-
        {{("t_out="+"%0.1f" |
        format(states('sensor.ble_temperature_bt_outside')|float))+
        "&t_kitchen="+("%0.1f" |
        format(states('sensor.ble_temperature_bt_indoor')|float)) +
        "&co2_bedroom="+states("sensor.workroom_co2")+
        "&note="+states("input_text.maker_badge_message")}}
    target:
      entity_id: input_text.zivy_obraz_hodnoty
  - service: rest_command.zivy_obraz_post
    data: {}
mode: single
```
5. Skontoluj či text `input_text.zivy_obraz_hodnoty` vyzerá dobre. Mal by mať formát `meno=hodnota1&meno2=hodnota2`. Ak potrebuješ doladiť formát, odporúčam skopírovať value do Debug Tools/Template a upravovať pokiaľ to nevyzerá dobre.
6. Hodnoty by sa mali aktualizovať na zivyobraz.eu. Uprav periódu aktualizáie a dáta podľa potreby.

    Dáta na zivyobraz.eu:
![image](https://github.com/Yourigh/zivy_obraz_HA/assets/25552139/a78fae86-1caa-4d52-8f44-84ce59b0f411)


HA Automatizácia v GUI
![image](https://github.com/Yourigh/zivy_obraz_HA/assets/25552139/ab873354-6161-401a-ac8d-8c058d8f9bf8)
