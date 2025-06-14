---
title: Weekend house network rebuild
date: 2024-06-13
tags:
  - network
---

Minulou středu jsme s kamarádem Ondrou provedli zásadní obnovu sítě na naší chalupě.

Cíle byly:
1. Propojit dvě budovy optickým spojem
2. Nainstalovat LTE anténu pro lepší připojení k internetu

Další cíle jsou:
4. Rozsegmentovat síť do více VLAN a přeřadit stávajícím síťovým a IoT zařízením nové adresy
5. Nahradit dvě stará SSID jedním, aby bylo možné plynulé přecházení po celém pozemku
6. Nastavit nová pravidla firewallu pro zabezpečení infrastruktury a povolit pouze nezbytný provoz

<!--more-->

## Optický spoj
Budovy jsou od sebe jen 5 metrů daleko, takže jsme použili hotový single-mode armovaný kabel. Na každé stěně jsou kotvy, které umožňují kabelu trochu pohybu. Instalovali jsme ho co nejvýše, takže je téměř neviditelný a v bezpečí před vysokými auty.

Na malé chalupě kabel pohodlně vede podél stěn do racku Mikrotik SR-10U, kde je připojen k hlavnímu routeru Mikrotik RB5009. Je zde dostatek místa pro servis zařízení a zároveň je zde i hlavní LTE uplink.

Na větší chalupě je kabel přichycen k trámu a později protažen chráničkou do vestavěné krabice, kde jsou umístěny Mikrotik CRS112 a CSS610.

## Připojení k Internetu
Dříve jsme byli připojeni 5GHz ac spojem od místního velkého ISP, ale kvůli odlehlé poloze je maximální rychlost, kterou pro koncové uživatele nabízí, 30/3 Mbps. Celá vesnice je připojena 60G spojem. Proto jsme hledali jiné možnosti a zatím jsou tři:

1) Firemní připojení od tohoto ISP přes dedikovaný 60G PtP spoj: Ideální řešení, ale zhruba sedmkrát dražší.
2) Starlink: Zvažoval jsem Starlink a jsem rád, že jsem ho nevybral, protože by byl třikrát pomalejší a třikrát dražší LTE.
3) LTE: Rozhodli jsme se pro LTE, protože to bylo něco, co jsme mohli vyzkoušet a případně nainstalovat natrvalo nebo vrátit. Bylo to dost riskantní, protože pokrytí signálem je u nás velmi špatné (měřeno z mobilu) a poskytovatelé zde službu "bezdrátový internet domů" nenabízí.

### Získání SIM karty pro LTE anténu

Před měsícem mě kontaktovalo externí call centrum zastupující T-Mobile. Nabídli mi slušnou cenu za SIM kartu s neomezenými daty. Když jsem se ji rozhodl pořídit, zavolal jsem přímo na T-Mobile a dostal jsem jinou, tentokrát dvakrát dražší nabídku. Operátor byl se mnou rychle hotový a protože už mám u T-Mobile firemní účet s několika vyššími službami, byl jsem dost zklamaný.

Naštěstí jsem to zkusil znovu a tentokrát jsem měl štěstí. Navštívil jsem pobočku T-Mobile v obchodním centru Nový Smíchov, kde si velmi příjemná Valerie dala práci s prostudováním mého účtu a společně jsme probrali různé možnosti. Nakonec jsem získal SIM kartu s neomezenou rychlostí a neomezenými daty za velmi slušnou cenu.

Z GSMWeb.cz jsme věděli, že na Buchtově kopci je velký vysílač – takže pravděpodobně se záložním napájením a optickým uplinkem. Je vzdálený 5 kilometrů a máme na něj přímou viditelnost ze střechy. Po úspěšném prvním pokusu ze země jsme anténu namontovali na střechu. Rychlost je prozatím stabilně ~350 Mbps dolů a ~50 Mbps nahoru, latence je horší, kolem ~50 ms. Signál je zatím velmi stabilní; měříme RSSI ~46 dBm a RSRP ~75 dBm. Uvidíme, co přinese déšť, bouřky a zima.

## Segmentace sítě

## Jedno hlavní SSID

## Firewall
