# IHW4
Артемьев Александр БПИ227(13 вариант).  
Программа разработана на языке программирования C++ с использованием для работы с потоками функций POSIX Thread.
# Условие:
Первая задача о магазине.
В магазине работают три отдела, каждый отдел обслуживает один продавец. Покупатель, зайдя в магазин, делает покупки в одном или нескольких произвольных отделах, обходя их в произвольном (случайном) порядке. Если в выбранном отделе продавец не свободен, покупатель становится в очередь и ожидает, пока продавец не освободится. Создать многопоточное приложение, моделирующее рабочий день магазина. Каждого покупателя и продавцов моделировать отдельными потоками. Размер очереди не оговаривается. Считается, что для данной задачи она не ограничена (но моделирование должно быть в разумных пределах).
# 4–5 баллов
• Соблюдены общие требования к отчету.  
• В отчете должен быть приведен сценарий, описывающий одновременное поведение представленных в условии задания сущностей в терминах предметной области. То есть, описан сценарий, задающий ролевое поведение субъектов и объектов задачи (интерпрета- ция условия с большей степенью детализации происходящего), а не то, как это будет реализовано в программе.  
• Описана модель параллельных вычислений, используемая при разработке многопоточной программы.  
• Описаны входные данные программы, включающие вариативные диапазоны, возможные при многократных запусках.  
• Реализовано консольное приложение, решающее поставленную задачу с использованием одного варианта изученных синхропримитивов.  
• Ввод данных в приложение реализован с консоли во время выполнения программы (без использования аргументов командной строки).  
• Для используемых генераторов случайных чисел описаны их диапазоны и то, как интерпретируются данные этих генераторов.  
• Вывод программы должен быть информативным,отражая все ключевые протекающие в ней события в терминах предметной области. Наблюдатель на основе вывода программы должен понимать, что в ней происходит в каждый момент времени ее работы.  
• В программе присутствуют комментарии, поясняющие выполняемые действия и описание используемых объектов и переменных.  
• Результаты работы программы представлены в отчете.  
# Сценарий  
В задаче 2 сущности: покупатель и отдел. Покупатель приходит в магазин и выбирает какие отделы он хотел бы посетить, то есть 1, 2, или 3 продавцов( 0 нерассматривается, так как с точки зрения задачи такой человек не встает в очередь и не производит оплаты, и бессмысленно его учитывать). Для каждого покупателя свой порядок, в котором он будет обходить эти отделы, прийдя в отдел он встает в очередь и ждет пока она не дойдет до него( либо его рассчитывают сразу если она была пуста), после чего идет в следующий отдел из своего списка. Обойдя все отделы, он уходит из магазина и не встречается больше в нашей модели. Продавец стоит в отделе за кассой и ждет когда к нему кто нибудь подойдет, если же перед ним очередь из людей он по порядку рассчитывает их, пока очередь не кончится. Очереди друг от друга никак не зависит, принято, что в каждой свой продавец и они работают параллельно, покупатели тоже друг от друга не зависят(только если они не в одной очереди, тогда один ждет другого).  
# Реализация  
С точки зрения реализации я создал два класса для покупатели и отдела Customer и Departments соответственно.
<img width="545" alt="Снимок экрана 2023-12-22 в 01 35 48" src="https://github.com/Vealar/IHW4/assets/121261496/202d4b52-776f-4b0a-ad41-16a454281943">
<img width="412" alt="Снимок экрана 2023-12-21 в 23 43 36" src="https://github.com/Vealar/IHW4/assets/121261496/2cc7f806-0e91-47bf-a012-df6cdefc8b7d">
Разберем класс Departments:  
Поле id идентефицирует отдел, то есть значение 0 1 2( вообще решение не зависит от количества отделов, можно поменять макрос COUNT_DEPARTMENTS)  
Статическое поле count_customers обозначает количество покупателей во всем магазине  
queue структура данных очередь в которой будут хранится покупатели, в этом отделе в нужном порядке  
mutex для каждой очереди свой, то есть работа в очередях не зависят друг от друга, и сейчас либо расплачивается клиент либо в очередь добавляется человек  
Конструктор и Деструктор для создания и удаления экземпляров класса  
isEmpty проверяет есть ли человек в очереди  
threadDepartments функция которая будет передаваться в поток очереди  
С точки зрения реализации методов, хочется объяснить threadDepartments

<img width="911" alt="Снимок экрана 2023-12-22 в 01 36 52" src="https://github.com/Vealar/IHW4/assets/121261496/c20da21a-baf8-4a7b-9d93-506a260dbe7d">

Поток будет работать пока в магазине есть покупатели. Если очередь не пуста, мы блокируем queue для работы с ней, расплачиваем покупателя, удаляем его из очереди и выводим сообщение об успехе операции, после чего отпускаем его из очереди и queue разблокирована. Здесь кроме блокирование самой queue и также есть глобальный MUTEX, который используется для блокировки консоли(без него потоки накладывают сообщения друг на друга)  
Разберем класс Customer:  
В целом коментарии позволяют понять, что и как используется.  
std::condition_variable cv: предназначен для синхронизации потоков. std::condition_variable позволяет потоку засыпать (выполнять wait) до тех пор, пока не получит сигнал о продолжении выполнения (выполнять notify_one или notify_all).  
std::mutex mutex_access: используется для обеспечения мутуального исключения доступа к общим данным из разных потоков. В данном контексте флаг commandReceived будет защищен от одновременного доступа из разных потоков.  
bool commandReceived = false Это флаг, используемый для синхронизации между потоками. По умолчанию он имеет значение false, но при условии что человека обслужили в данном отделе поток отдела изменяет его значение на true. Поток покупателя ждет, пока commandReceived не станет true, после чего продолжает выполнение.  
<img width="911" alt="Снимок экрана 2023-12-22 в 01 51 33" src="https://github.com/Vealar/IHW4/assets/121261496/fadaeb50-e04b-4351-a1a7-48216db7290a">
Посмотрим на конструктор: мы генерируем количество отделов, которые хочет посетить человек.  
Берем вектор order, состоящий из всех отделов, перемешиваем его элементы в рандомном порядке и оставляем только count_departments из них  .Следующий метод threadCustomer
<img width="911" alt="Снимок экрана 2023-12-22 в 01 53 32" src="https://github.com/Vealar/IHW4/assets/121261496/de52ad14-d0b1-4e31-bc0f-81e7a3cc03a3">  
Метод, который подается в поток покупателя. Покупатель встает в очередь для этого блочит ее mutex. Также блочит консоль, чтобы вывести сообщение. И ожидает когда его обслужат.  
Последний элемент это main  
<img width="904" alt="Снимок экрана 2023-12-22 в 00 08 12" src="https://github.com/Vealar/IHW4/assets/121261496/a83fb346-900f-4a56-9e3a-7aec9655158b">
Здесь мы создаем поток под каждого покупателя и каждый отдел, они отрабатывают и мы их закрываем. Пользователь передает количество покупателей через консоль, число int. В программе использованы примитивы синхронизации такой, как mutex. Данные которые генерируются в задаче это количество отделов int число от 1 до 3. Теперь предоставлю тесты при n = 0,1,5,10:
<img width="332" alt="Снимок экрана 2023-12-22 в 01 34 14" src="https://github.com/Vealar/IHW4/assets/121261496/141efd21-537e-4674-ab83-62e93552943c">  
<img width="332" alt="Снимок экрана 2023-12-22 в 01 34 20" src="https://github.com/Vealar/IHW4/assets/121261496/a2156822-0cbd-413a-a800-49e723f25075">  
<img width="345" alt="Снимок экрана 2023-12-22 в 02 03 15" src="https://github.com/Vealar/IHW4/assets/121261496/b80672b5-e681-4eca-8e48-298d551560b0">  
<img width="274" alt="Снимок экрана 2023-12-22 в 00 20 01" src="https://github.com/Vealar/IHW4/assets/121261496/c106d477-8d29-4c5d-9586-cd813b5401d5">  
<img width="332" alt="Снимок экрана 2023-12-22 в 01 24 17" src="https://github.com/Vealar/IHW4/assets/121261496/2eb9998c-6a63-446e-9917-c0facccfe3ea"> 
# 6 - 7 баллов  
В дополнение к требованиям на предыдущую оценку и, в некоторых случаях вместо них:  
• В отчете подробно описан обобщенный алгоритм, используемый при реализации программы исходного словесного сценария. В котором показано, как на программу отображается каждый из субъектов предметной области.  
• В программу добавлена генерация случайных данных в допустимых диапазонах.  
• Реализован ввод исходных данных из командной строки при запуске программы вместо ввода параметров с консоли во время выполнения программы.  
• Результаты изменений отражены в отчете.  
Алгоритм был достаточно подробно описан в пункте на 4 - 5 баллов. Если собрать все вместе: Покупатель реализуется как класс, имеющей mutex_access для каждой очереди и отдел реализуется как класс, имеющий свой личный mutex. Также у всех сущностей есть доступ к полю MUTEX, который будет блокировать консоль для вывода сообщения конкретной сущностью. Отдел блокирует очередь для обслуживание покупателя и покупатель может блокировать очередь для того чтобы попасть в нее, все это сделано через поле mutex для конкретного отдела. Когда покупатель стоит и ожидает в очереди mutex_access выступает в роли метода для ожидания, он реализован с помощью condition_variable cv и commandReceived. Когда покупатель приходит к продавцу флаг = true и отдел выводит сообщение с задержкой 0,5 секунды(на обслуживание). После чего покупатель идет в другой отдел. ПЕремещение покупателей происходит паралельно, так же как и работа очередей. В конце когда все покупатели обошли все отделы count_customers = 0, и все потоки завершаются.  
Генерация случаный данных была и описна в Customer::random()  
Реализован ввод через командную строку:
<img width="898" alt="Снимок экрана 2023-12-22 в 03 15 19" src="https://github.com/Vealar/IHW4/assets/121261496/a44b299d-3170-4042-9596-91e8a23bb611">  
Поменял начало main, пример для n = 3, n = 5
<img width="663" alt="Снимок экрана 2023-12-22 в 03 17 49" src="https://github.com/Vealar/IHW4/assets/121261496/49749ebb-ef08-40fe-9d8f-f6406c672823">
<img width="663" alt="Снимок экрана 2023-12-22 в 03 18 56" src="https://github.com/Vealar/IHW4/assets/121261496/e2835b30-b4fe-494f-9ab7-a758dd0891b3">  
# 8 баллов
В дополнение к требованиям на предыдущую оценку
• В программу, наряду с выводом в консоль, добавлен вывод результатов в файл. Имя файла для вывода данных задается в командной строке как один из ее параметров.  
• Результаты работы программы должны выводиться на экран и записываться в файл.  
• Наряду с вводом исходных данных через командную строку добавлен альтернативный вариант их ввода из файла, который по сути играет роль конфигурационного файла. Имя этого файла задается в командной строке вместо параметров, которые в этом случае не вводятся. Управление вводом в командной строке осуществляется с использованием соответствующих ключей.  
• Приведено не менее трех вариантов входных и выходных файлов с различными исходным данными и результатами выполнения программы.  
• Ввод данных из командной строки расширен с учетом введенных изменений.  
• Результаты изменений отражены в отчете  
существенно изменился main:
<img width="802" alt="Снимок экрана 2023-12-22 в 03 48 56" src="https://github.com/Vealar/IHW4/assets/121261496/97d3fbd5-5019-4881-9429-a19bc63438bf">  
Я добавил статическое поле potok в класс покупателя, туда после чтение данных из командной строки будет записывать поток записи, и вместе с выводом в консоль выводится в файл. В майне, происходит проверка число ли передано в консоль, если да то это количество покупателей и следующий элемент имя файла, иначе мы читаем данные с фала, который в командной строке. Далее привожу 3 теста.  
1) 5 output.txt 2)output.txt 3)11 out.txt
<img width="663" alt="Снимок экрана 2023-12-22 в 03 47 53" src="https://github.com/Vealar/IHW4/assets/121261496/c3900362-60de-44cd-a1d4-2e5c51507511">
<img width="1129" alt="Снимок экрана 2023-12-22 в 03 55 20" src="https://github.com/Vealar/IHW4/assets/121261496/836cb68e-5f15-4e00-a9ad-89fbfa0cb721">
<img width="1129" alt="Снимок экрана 2023-12-22 в 03 55 27" src="https://github.com/Vealar/IHW4/assets/121261496/a17baf5d-84cb-48a5-bd35-1a1ab7cd7fd3">
<img width="1129" alt="Снимок экрана 2023-12-22 в 03 56 32" src="https://github.com/Vealar/IHW4/assets/121261496/ecf2fcf1-8307-4065-ad2b-5a77b516c2ed">


