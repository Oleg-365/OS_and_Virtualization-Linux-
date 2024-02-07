Задание

• Написать скрипт очистки директорий.
На вход принимает путь к директории.
Если директория существует, то удаляет в ней все файлы с расширениями .bak, .tmp, .backup.
Если директории нет, то выводит ошибку.


Решение.

Создаем и открываем файл следующей командой:

  nano delfiletodir

Далее прописываем текст:

#!/bin/bash

read -p "Введите путь к дирректории: " DELDIR

if [ -e $DELDIR ]
    then
        echo 'Указанная дирректория найдена'
        cd $DELDIR
        echo 'Произвожу удаление'
        rm -v *.bak *.tmp *.backup
        echo 'Удаление завершено успешно'
    else
        echo 'Указанная дирректория не найдена! Выполнение остановлено'
        exit 2
fi

Сохраняем и запускаем в Bash:

bash delfiletodir


1)Создал файл crontab, который ежедневно регистрирует занятое каждым пользователем дисковое пространств.
        Если взять за основу что каждый пользователь держит свои данные в /home/$USER то запись в кронтабе будет выглядеть так
        * 1 * * * du -sm /home/* > /var/log/user_data_size.log
 
        Создать скрипт ownersort.sh, который в заданной папке копирует файлы в директории, названные по имени владельца каждого файла. Учтите, что файл должен остаться принадлежать соответствующему владельцу.
        
#!/bin/bash
 
 
dir="$1"
 
script=`echo $0 | xargs basename`
 
        for name_file in $dir/* ;
 
        do
 
        if [[ `basename $name_file` != 'ownersort.sh' ]] ; then
 
                user_name=`ls -la $name_file | awk {'print $3; '} | sort | uniq | xargs echo`
 
                mkdir -p $dir/$user_name && chown -vR $user_name $dir
 
                dirname=($dir/$user_name)
 
                cp -pv $name_file $dirname
 
                rm -v $name_file
 
                echo
                echo "Copy done"
         else
 
         echo
         echo "This is a script file. Skip file"
 
 
        fi
 
done    
        
        Обязательным условием должно быть выполнение с правами root, иначе сохранение владельца не получится, будет записаны файлы с правами текущего пользователя.
        2. Написал скрипт rename.sh, аналогичный разобранному, но порядковые номера файлов выравнивать, заполняя слева нуля до ширины максимального значения индекса. Т.е. newname000.jpg newname102.jpg (Использовать printf)
       
#!/bin/bash
 
dir=$1
 
        for name_file in $dir/* ; do
 
        f_file=`basename $name_file `
 
        if [[ $f_file != 'rename.sh' ]] ; then
 
                f_new_name=`printf 'IMG_%03d.jpg\n' "$f_file"`
 
                echo
                echo "File is $f_file rename to $f_new_name "
 
                mv -v $name_file $dir/$f_new_name
 
                echo "Rename file done"
 
            else
 
             echo
             echo "No rename file. This is a script file."
 
        fi
 
done
 
        4. *Написать скрипт резервного копирования по расписанию следующим образом:
        В первый день месяца помещать копию в backdir/montlhy
        Бэкап по пятницам хранить в каталоге backdir/weekley
        В остальные дни сохранять копии в backdir/daily
        Настроить ротацию следующим образом. Ежемесячные копии хранить 180 дней, ежедневные — неделю, еженедельные — 30 дней. Подсказка: для ротации используйте find.
 
#!/bin/bash
 
opt=$1
dir=$2
b_dir="/mnt/repo/Developer/Edu/OS/Linux/DZ/4/script/backup"
 
dt=`date +%d-%m-%Y`
 
if [[ $opt = 'day' ]] ; then
 
        echo 
        echo "Days backup"
 
        tar -cvjpf $b_dir/daily/$dt.tar.bz2 $dir/*
 
        size_d_backup=$(du -s $b_dir/daily/$dt.tar.bz2 | awk {'print $1; '})
 
        echo
        echo "Days backup SIZE = $size_d_backup"
        echo "Backup done"
 
     else
 
        echo
 
fi
 
if [[ $opt = 'week' ]] ; then
 
        echo 
        echo "Week backup"
 
        mv -v $b_dir/daily/*.tar.bz2 $b_dir/weekley/
 
        rm -v $b_dir/daily/*
 
        size_w_backup=$(du -s $b_dir/weekley | awk {'print $1; '})
 
        echo
        echo "Week backup SIZE = $size_w_backup"
        echo "Week backup done"
 
      else
 
        echo
fi
if [[ $opt = 'month' ]] ; then
 
        echo
        echo "Month backup"
 
        mkdir -pv $b_dir/montlhy/`date +%m`
 
        mv -v $b_dir/weekley/*.tar.bz2 $b_dir/montlhy/`date +%m`/
 
        rm -v $b_dir/weekley/*
 
        size_m_backup=$(du -sm $b_dir/montlhy/`date +%m` | awk {'print $1; '})
 
        echo
        echo "Month backup SIZE = $size_m_backup"
        echo "Month backup done"
 
      else
        echo
fi
 
if [[ $opt = 'month_clean' ]] ; then
 
        echo
        echo "Month backup clean"
        rm -vR $b_dir/montlhy/*
 
        echo
        echo "Month backup clean done"
        echo
 
      else
        echo
 
fi
 
if [[ $opt = 'help' ]] ; then
 
        echo
        echo "Script from backup file"
        echo
        echo "$0        Options Dir"
        echo 
        echo "          Options:  "
        echo "                  day     -       Days backup"
        echo "                  week    -       Week backup"
        echo
        echo "                  Dir     -       Directory file from backup"
        echo
 
      else
        echo
 
fi
 
Единственное целесообразность очистки по месяцам надо до конца обдумать. Либо тогда уже вычислять что 3х месячные оставлять а остальные в топку.
