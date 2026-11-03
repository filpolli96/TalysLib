# Журнал работы: TalysLib / TALYS / ROOT

Подробности по ошибкам сборки и их исправлениям: [TalysLib_fixes_new_ROOT.md](TalysLib_fixes_new_ROOT.md).

## 03.11.2026

### Git и резервная копия TalysLib

- Fork: https://github.com/filpolli96/TalysLib (оригинал: https://github.com/terawatt93/TalysLib).
- В fork'е был мой коммит от 25.05.2026 `6cefe10` *"Add proton OMP support and TalysFitterMT improvements for p+12C fitting"*
  (`source/TalysFitterMT.cc`, `source/LevelProperties.cc`, `TalysLib.hh`).
- Локальная папка `~/Programs/TalysLib` переподключена:
  - `origin` = мой fork `filpolli96/TalysLib`, сюда делаю push;
  - `upstream` = `terawatt93/TalysLib`, отсюда беру обновления.
- Мой коммит объединён с 6 новыми коммитами terawatt93 (май–октябрь), конфликтов не было.
  Результат (`9b0bcf8`) отправлен в fork через VS Code → Sync Changes.
- git настроен: `user.name = filpolli96`, `user.email = filpolli96@gmail.com`.
- Вход в GitHub: через VS Code (Source Control → Sync, авторизация в браузере). SSH-ключ не настраивала.

**Как работать дальше:**
```bash
git add -A && git commit -m "что сделала" && git push   # сохранить свои изменения
git pull upstream main && git push                      # забрать обновления terawatt93
```
Восстановление после поломки диска: `git clone https://github.com/filpolli96/TalysLib.git`.

### Сборка TalysLib с ROOT 6.40.04

1. `source/TalysFitter.cc:79`: убраны лишние аргументы `"TalysFitter","Eval"` у конструктора `TF1`.
2. ROOTOpenXLSX при первой сборке получилась без кода OpenXLSX (ошибка в её Makefile),
   из-за этого `TalysLib.so` не загружалась (`undefined symbol ... XLCellAssignable`).
   Помогла пересборка: `make clean && make`. Правильная `ROOTOpenXLSXLib.so` весит около 12 МБ.
3. Папки ROOT перенесены в `~/Programs/root/` (`root_src`, `root_build`, `root_install`).
4. `~/.bashrc`: ROOT подключается по полному пути
   `source ~/Programs/root/root_install/bin/thisroot.sh`, повторы `export ... TalysLib` удалены.

Проверено: TalysLib загружается в ROOT из нового терминала, `Nucleus("12C")` создаётся.

### Переменная TALYSVERSION

- Должна быть в `~/.bashrc`, например `export TALYSVERSION=2`. Без неё
  `source/NucleiProperties.cc:2077` (`atof(getenv("TALYSVERSION"))`) падает
  с segfault при расчёте.
- Дробное значение допустимо, но меняет поведение:
  - `Parser.cpp` (`GetTalysVersion`, `atoi`): `2` и `2.2` дают одно и то же, 2;
  - `NucleiProperties.cc:2077` (`atof(...) > 2`): при `2.2` включается другая ветка расчёта
    (`ExecuteCalculationInTalys` + `GenerateListOfProducts_v2`, расчёт сетки энергий отдельными ядрами).
- `Install.sh` ставит `2`.
- Установленный TALYS (`~/Programs/talys`) свежий: версия 2.2 или новее, последний коммит от 01.11.2026.

## Задачи

- [ ] **Сравнить расчёты при разных версиях TALYS / значениях `TALYSVERSION`.**
  Посчитать одну и ту же реакцию (например, p+¹²C) при `TALYSVERSION=2` и `TALYSVERSION=2.2`
  и сравнить сечения. Сравнить с расчётом в другой версии TALYS (например, в `talys_mod`).
  Спросить terawatt93, какое значение правильное для TALYS 2.2 и рабочая ли ветка `> 2`.
- [ ] Вернуть `export TALYSVERSION=2` в `~/.bashrc`, если ещё не сделано.
- [ ] Закоммитить и отправить в fork исправление `TalysFitter.cc` и заметки (`*.md`).
- [ ] Сохранить `talys_mod` на GitHub: fork https://github.com/arjankoning1/talys, отдельная ветка
  с моими правками (10 изменённых файлов + `source/patch.py`). Сейчас они есть **только на диске**.
- [ ] (по желанию) Сохранить правку `source/machine.f90` в `~/Programs/talys`.
- [ ] (по желанию) Предложить авторам исправление `TF1` (pull request в terawatt93/TalysLib)
  и сообщить об ошибке в Makefile ROOTOpenXLSX.
