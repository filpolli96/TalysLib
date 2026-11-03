# TalysLib: что исправить для сборки с новым ROOT

Репозиторий: https://github.com/terawatt93/TalysLib
Система: Ubuntu 24.04, gfortran 13, свежий ROOT 6.x.

## 1. Окружение ROOT перед сборкой (не правка кода)

**Симптом:**
```
make: rootcling: No such file or directory
make: *** [Makefile:14: TalysLibDict.cxx] Error 127
```

**Причина:** ROOT не подключён в текущем терминале, поэтому `rootcling` нет в `PATH`.

**Решение:**
```bash
source /путь/к/root/bin/thisroot.sh
which rootcling      # должен показать путь
./Install.sh
```
Для постоянного подключения добавьте строку `source .../thisroot.sh` в **конец** `~/.bashrc` и откройте новый терминал.

## 2. Устаревший конструктор `TF1` (обязательная правка)

**Файл:** `source/TalysFitter.cc`, строка 79, функция `TalysFitter::GenerateTF1`.

**Симптом:**
```
error: no matching function for call to ‘TF1::TF1(const char*, TalysFitter*,
double (TalysFitter::*)(double*, double*), double&, double&,
std::vector<double>::size_type, const char [12], const char [5])’
```

**Причина:** в новых версиях ROOT у конструктора `TF1` для функции-члена класса убраны
последние аргументы с именем класса и именем метода (`"TalysFitter"`, `"Eval"`).
Они и раньше не использовались. Теперь на этих позициях стоят другие параметры
(`ndim` и т.д.), и строки туда не подходят.

**Было:**
```cpp
FitFunction=TF1(name.c_str(),this,&TalysFitter::Eval,x_min,x_max,Parameters.size(),"TalysFitter","Eval");
```

**Стало:**
```cpp
FitFunction=TF1(name.c_str(),this,&TalysFitter::Eval,x_min,x_max,Parameters.size());
```

**Командой** (из корня TalysLib):
```bash
sed -i 's/,"TalysFitter","Eval")/)/' source/TalysFitter.cc
```

Поведение функции не меняется. Другие вызовы `TF1` с этими лишними аргументами
в репозитории не найдены (проверено по `*.cc`, `*.hh`, `*.cxx`, `*.h`).

## 3. Мелочи (не мешают сборке на Ubuntu x86_64)

- **`Install.sh` ставит несуществующий пакет `libsqlite-dev`.** Ошибка
  `Unable to locate package libsqlite-dev` безвредна: следом скрипт ставит
  правильный `libsqlite3-dev`.
- **В `Makefile` (строка 19) пути к библиотекам захардкожены:**
  `/usr/lib/x86_64-linux-gnu/libzip.so`, `/usr/lib/x86_64-linux-gnu/libsqlite3.so`.
  На Ubuntu x86_64 они верные. На другой системе или архитектуре их можно
  заменить на `-lzip -lsqlite3`.
- **Предупреждения компилятора** (например, `variable ‘MinEnergy’ set but not used`
  в `source/NucleiProperties.cc`) на сборку не влияют.

## 4. Альтернатива правкам кода

Можно поставить рядом более старый ROOT (например, 6.28 или 6.30) из бинарного архива
с https://root.cern/install/ и перед сборкой выполнить `source` его `thisroot.sh`.

## 5. ROOTOpenXLSX собрана без OpenXLSX (TalysLib.so не загружается)

**Симптом:** сборка TalysLib проходит, но при загрузке в ROOT:
```
Error in <TCling__LazyFunctionCreatorAutoloadForModule>: Failed to load library .../ROOTOpenXLSXLib.so
undefined symbol: _ZTVN8OpenXLSX16XLCellAssignableE
```

**Причина:** ошибка в `Makefile` ROOTOpenXLSX. Список исходников OpenXLSX
(`find ... -name "*.cpp"`) вычисляется при старте `make`, а сами исходники
скачиваются (`git submodule update`) уже позже, внутри правила для словаря.
При самом первом `make` папка `external/OpenXLSX` пустая, и библиотека
собирается без OpenXLSX (размер ~1.7 МБ вместо ~12 МБ).

**Решение:** просто пересобрать ROOTOpenXLSX ещё раз (исходники уже скачаны):
```bash
cd ~/Programs/ROOTOpenXLSX
make clean && make
```
Проверка: `ls -la ROOTOpenXLSXLib.so` — около 12 МБ.

## 6. Относительный путь к ROOT в ~/.bashrc

Строка `source ../root_install/bin/thisroot.sh` работает, только если терминал
открыт в папке рядом с `root_install`. Нужен полный путь:
```bash
source /home/polina/Programs/root_install/bin/thisroot.sh
```
(`make install` в TalysLib каждый раз дописывает в `~/.bashrc` три строки
`export ... TalysLib` — повторы можно удалить.)

## Итог (03.11.2026, ROOT 6.40.04)

Пункты 2 и 5 исправлены, TalysLib собрана и загружается в ROOT
(классы `Nucleus`, `TalysFitter`, `TalysFitterMT`, `TLMaterial` доступны).

## 7. Если появятся новые ошибки

Записать сюда текст ошибки, файл и строку, а также вывод `root-config --version`.
Правки стоит отправить авторам в виде issue или pull request.
