# Домашнее задание к занятию "4.1. Командная оболочка Bash: Практические навыки"
1. Переменной c будет присвоено значение "a+b" т.к. обращение к переменным не идет, это просто строковое значение.
Переменной d будет присвоено значение "1+2" т.к. из-за знака "+" переменные a и b интерпретируются как строки. 
Переменной e будет присвоено значение "3" т.к. конструкция $((   )) указывает что это арифметическая операция.

2. Скрипт
```bash
while ((1==1)
  do
  curl https://localhost:4757
  if (($? != 0))
  then
    date >> curl.log
  fi
done
```

2.1 В первую очередь в скрипте есть ошибка которая не дает ему выполниться. Нет закрывающей скобки в операторе while. Исправленный вариант:
```bash
while ((1==1))
  do
  curl https://localhost:4757
  if (($? != 0))
  then
    date >> curl.log
  fi
done
```

2.2 В задаче написано что при работе скрипта потребяется место на диске. Место потребляется только если проблема продолжается. Если нужно знать только последнюю отметку времени то можно модифицировать скрипт и заменить оператор добавления текста на замену текста:
```bash
while ((1==1))
  do
  curl https://localhost:4757
  if (($? != 0))
  then
    date > curl.log
  fi
done
```

2.3 Если нам нужно чтобы скрипт завершал свою работу после того как пробелма пропала, можно джоработать его добавив выход по успешному выполнению curl:
```bash
while ((1==1))
  do
  curl https://localhost:4757
  if (($? != 0))
  then
    date >> curl.log
  else
    break
  fi
done
```

3. Скрипт проверки доступности хостов по порту 80
```bash
#!/bin/bash

logfile="out.log"
port=80
hosts=("192.168.0.1" "173.194.222.113" "87.250.250.242")

for i in {0..4}
do
  for host in ${hosts[@]}
  do
    nc -z -w1 $host $port
    rc=$?
    if (($rc == 0))
    then
      echo `date` $host OK >> $logfile
    else
      echo `date` $host ERROR >> $logfile
    fi
  done
done
```

4. Модификация скипта "Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается"
```bash
#!/bin/bash

logfile="out.log"
errfile="error"
port=80
hosts=("192.168.0.1" "173.194.222.113" "87.250.250.242")

for i in {0..4}
do
  for host in ${hosts[@]}
  do
    nc -z -w1 $host $port
    rc=$?
    if (($rc == 0))
    then
      echo `date` $host OK >> $logfile
    else
      echo `date` $host ERROR >> $logfile
      echo $host >> $errfile
      exit 1
    fi
  done
done
```

5. Пример hook-а для Git.
Хук идет через файл .git/hooks/commit-msg
Содержимое файла:
```bash
#!/bin/bash

template="\[.*\]"
commitfile=$1
maxlength=30

length=`cat $commitfile | wc -c`
(grep -Eq $template $commitfile) && match=1 || match=0

if (($match == 0))
then
  echo >&2 Commit message not match template.
  err=1
fi

if (("$length" > "$maxlength"))
then
  echo >&2 Commit message too long. Max length $maxlength.
  err=1
fi

if ((err == 1))
then
  exit 1
fi
```

