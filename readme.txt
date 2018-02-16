## 导入所有必要的包和函数。
import csv # 读写 csv 文件
from datetime import datetime # 日期解析操作
from pprint import pprint # 用于输出字典等数据结构 这比 base print 函数要好用。

def print_first_point(filename):
    """
    本函数会输出并返回指定的 csv 文件 （含页眉行）的第一个数据点（即文件的第二行）。
    """
    # 输出城市名以供参考
    city = filename.split('-')[0].split('/')[-1]
    print('\nCity: {}'.format(city))
    
    with open(filename, 'r') as f_in:
        ## 待办：用 csv 库来设置一个 DictReader 对象。##
        ## 见 https://docs.python.org/3/library/csv.html           ##
        trip_reader = csv.DictReader(f_in)
        
        ## 待办：对 DictReader 对象使用函数     ##
        ## 从而读取数据文件的第一条骑行记录并将其存储为一个变量     ##
        ## 见 https://docs.python.org/3/library/csv.html#reader-objects ##
        first_trip = next(trip_reader)
        
        ## 待办：用 pprint 库来输出第一条骑行记录。 ##
        ## 见 https://docs.python.org/3/library/pprint.html     ##
        pprint(first_trip)
        
    # 输出城市名和第一条骑行记录以备测试
    return (city, first_trip)

# 各城市的文件列表
data_files = ['./data/NYC-CitiBike-2016.csv',
              './data/Chicago-Divvy-2016.csv',
              './data/Washington-CapitalBikeshare-2016.csv',]


# 输出各文件的第一条骑行记录，并将其储存在字典中
example_trips = {}
for data_file in data_files:
    city, first_trip = print_first_point(data_file)
    example_trips[city] = first_trip

def duration_in_mins(datum, city):
    """
    将一个字典作为输入，该字典需包含一条骑行记录（数据）
    及记录城市（城市）的信息，返回该骑行的时长，使该时长以分钟为单位。
    
    记住，华盛顿特区是以毫秒作为计量单位的，而芝加哥和纽约市则
    以秒数作为单位。
    
    提示：csv 模块会将所有数据读取为字符串，包括数值，
    所以转换单位时，你需要用一个函数来将字符串转换为合适的数值类型。
    见 https://docs.python.org/3/library/functions.html
    """
    
    # 请在此处写出代码
    if city in ['NYC','Chicago']:
        duration_s = int(datum['tripduration'])
    elif city in ['Washington']:
        duration_s = int(datum['Duration (ms)'])/1000
    duration = duration_s/60
    return duration


# 测试代码是否奏效，若所有断言都没问题，则不应有输出出现。
# 至于字典 `example_trips` 
# 则是在你输出每个数据源文件的第一条骑行数据时生成的。
tests = {'NYC': 13.9833,
         'Chicago': 15.4333,
         'Washington': 7.1231}

for city in tests:
    assert abs(duration_in_mins(example_trips[city], city) - tests[city]) < .001

def time_of_trip(datum, city):
    """
    将一个字典作为输入，该字典需包含一条骑行记录（数据）
    及记录城市（城市）的信息，返回该骑行进行的月份、小时及周几这三个值。
    
    
    记住，纽约市以秒为单位，华盛顿特区和芝加哥则不然。
    
    提示：你需要用 datetime 模块来将原始日期字符串解析为
    方便提取目的信息的格式。
    见 https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior
    """
    
    # 请在此处写出代码
    if city in ['NYC']:
        start_time = datetime.strptime(datum['starttime'], '%m/%d/%Y %H:%M:%S')
    elif city in ['Chicago']:
        start_time = datetime.strptime(datum['starttime'], '%m/%d/%Y %H:%M')
    elif city in ['Washington']:
        start_time = datetime.strptime(datum['Start date'], '%m/%d/%Y %H:%M')
    month, hour, day_of_week = start_time.month, start_time.hour, start_time.strftime('%A')
    return (month, hour, day_of_week)


# 测试代码是否奏效，若所有断言都没问题，则不应有输出出现。
# 至于字典 `example_trips`
# 则是在你输出每个数据源文件的第一条骑行数据时生成的。
tests = {'NYC': (1, 0, 'Friday'),
         'Chicago': (3, 23, 'Thursday'),
         'Washington': (3, 22, 'Thursday')}

for city in tests:
    assert time_of_trip(example_trips[city], city) == tests[city]


def type_of_user(datum, city):
    """
    将一个字典作为输入，该字典需包含一条骑行记录（数据）
    及记录城市（城市）的信息，返回进行该骑行的系统用户类型。
    
    
    记住，华盛顿特区的类名与芝加哥和纽约市的不同。
    
    """
    format_user_type = {
        'Registered':'Subscriber',
        'Casual':'Customer'
    }
    
    # 请在此处写出代码
    if city in ['NYC','Chicago']:
        user_type = datum['usertype']
    elif city in ['Washington']:
        user_type = format_user_type[datum['Member Type']]
    return user_type


# 测试代码是否奏效，若所有断言都没问题，则不应有输出出现。
# 至于字典 `example_trips`
# 则是在你输出每个数据源文件的第一条骑行数据时生成的。
tests = {'NYC': 'Customer',
         'Chicago': 'Subscriber',
         'Washington': 'Subscriber'}

for city in tests:
    assert type_of_user(example_trips[city], city) == tests[city]

def condense_data(in_file, out_file, city):
    """
    本函数会从指定的输入文件中提取全部数据
    并在指定的输出文件中写出浓缩数据。
    城市参数决定输入文件的解析方式。
    
    提示：参考下框以明确参数结构！
    """
    
    with open(out_file, 'w') as f_out, open(in_file, 'r') as f_in:
        # 设置 csv DictWriter 对象——该对象需将第一列列名
        # 作为 "fieldnames" 参数
        out_colnames = ['duration', 'month', 'hour', 'day_of_week', 'user_type']        
        trip_writer = csv.DictWriter(f_out, fieldnames = out_colnames)
        trip_writer.writeheader()
        
        ## 待办：设置 csv DictReader 对象##
        trip_reader = csv.DictReader(f_in)

        # 收集并处理每行的数据
        for row in trip_reader:
            # 设置一个字典来存储清理和修剪后的数据点的值
            new_point = {}

            ## 待办：使用辅助函数来从原始数据字典中获取清理数据##
            ## 注意字典 new_point 的关键词应与 ##
            ## 上述 DictWriter 对象设置的列名一致。        ##
            new_point['duration'] = duration_in_mins(row, city)
            date_time = time_of_trip(row, city)
            new_point['month'] = date_time[0]
            new_point['hour'] = date_time[1]
            new_point['day_of_week'] = date_time[2]
            new_point['user_type'] = type_of_user(row, city)

            ## 待办：在输出文件中写出处理后的信息。##
            ## 见 https://docs.python.org/3/library/csv.html#writer-objects ##
            trip_writer.writerow(new_point)
            
            
# 运行下框以测试效果
city_info = {'Washington': {'in_file': './data/Washington-CapitalBikeshare-2016.csv',
                            'out_file': './data/Washington-2016-Summary.csv'},
             'Chicago': {'in_file': './data/Chicago-Divvy-2016.csv',
                         'out_file': './data/Chicago-2016-Summary.csv'},
             'NYC': {'in_file': './data/NYC-CitiBike-2016.csv',
                     'out_file': './data/NYC-2016-Summary.csv'}}

for city, filenames in city_info.items():
    condense_data(filenames['in_file'], filenames['out_file'], city)
    print_first_point(filenames['out_file'])

def number_of_trips(filename):
    """
    本函数会读取一个骑行数据文件，分别报告
    会员、散客和所有系统用户的骑行次数。
    """
    with open(filename, 'r') as f_in:
        # 设置 csv reader 对象
        reader = csv.DictReader(f_in)
        
        # 初始化计数变量
        n_subscribers = 0
        n_customers = 0
        # 计算骑行类型
        for row in reader:
            if row['user_type'] == 'Subscriber':
                n_subscribers += 1
            else:
                n_customers += 1
        
        # 统计骑行总次数
        n_total = n_subscribers + n_customers
        
        # 将结果作为数组返回出来
        return(n_subscribers, n_customers, n_total)

## 修改此框及上框，回答问题 4a。##
## 记得运行你在问题 3 中创建的数据文件清理函数。     ##

data_file = {'Washington': './data/Washington-2016-Summary.csv',
             'Chicago': './data/Chicago-2016-Summary.csv',
             'NYC': './data/NYC-2016-Summary.csv'}
number_dict = {}
for city, file_path in data_file.items():
    number_dict[city] = number_of_trips(file_path)
# 骑行次数最多的城市
print("骑行次数最多的城市是：" + max(number_dict,key=lambda x:number_dict[x][2]))
# 会员进行的骑行次数占比最高的城市
print("会员进行的骑行次数占比最高的城市：" + max(number_dict,key=lambda x:number_dict[x][0]/number_dict[x][2]))
# 散客进行的骑行次数占比最高的城市 
print("散客进行的骑行次数占比最高的城市是：" + max(number_dict,key=lambda x:number_dict[x][1]/number_dict[x][2]))

      
def filter_trip_user_type(lists,user_filter):
    """
    用户类型过滤
    Args:
        lists: list 骑行数据列表
        user_filter: string 用户类型
    Return:
        list 过滤后的列表
    """
    return list(filter(lambda x: x[4] == user_filter, lists))

def duration_of_trips(lists,user_filter=None):
    """
    获取骑行时长
    Args:
        lists: list 骑行数据列表
        user_filter: string 用户类型筛选，默认不筛选
    Return: 
        float 平均骑行时长
    """
    #骑行时间列表
    duration_list = []
    tmp_list = lists
    # 过滤用户类型
    if user_filter:
        tmp_list = filter_trip_user_type(lists,user_filter)
    duration_list = [float(x[0]) for x in tmp_list]
        
    return duration_list

def avg_duration_of_trips(lists,user_filter=None):
    """
    统计平均骑行时长
    Args:
        lists: list 骑行数据列表
        user_filter: string 用户类型筛选，默认不筛选
    Return: 
        float 平均骑行时长
    """
    duration_list = duration_of_trips(lists,user_filter)
        
    return sum(duration_list)/ len(duration_list)    

def over_mins_proportion_of_trips(lists,mins,user_filter=None):
    """
    统计收时间超过指定骑行的比例
    Args:
        lists: list 骑行数据列表
        mins: float 骑行时间
        user_filter: string 用户类型筛选，默认不筛选
    Return:
        string 超过骑行时间骑行的比例
    """
    # 初始化超过30分钟骑行列表
    over_mins_list = []
    tmp_lists = lists
    # 过滤用户类型
    if user_filter:
        tmp_lists = filter(lambda x: x[4] == user_filter, tmp_lists)
    # filter返回的是filter对象 需要用list转换一下
    over_mins_list = list(filter(lambda x: float(x[0]) > mins, tmp_lists))
    
    return len(over_mins_list) / len(lists)

# 测试用例  
# 不要运行
data_file = './examples/BayArea-Y3-Summary.csv'
with open(data_file,'r') as f_in:
    reader = csv.reader(f_in)
    bay_area_list = list(reader)
    bay_area_list = bay_area_list[1:]
print(over_mins_proportion_of_trips(bay_area_list,30.00))
print(avg_duration_of_trips(bay_area_list))

## 使用本框及新框来回答问题 4b。               ##
##                                                                      ##
## 提示：csv 模块会将所有数据读取为字符串，包括数值。 ##a
## 因此，在统计数据之前，你需要用函数将字符串转换为      ##
## 合适的数值类型。         ##
## 小贴士：在湾区示例数据中，平均骑行时长为 14 分钟，##
## 骑行时长多于 30 分钟的数据占比 3.5%。                      ##


data_file = {'Washington': './data/Washington-2016-Summary.csv',
             'Chicago': './data/Chicago-2016-Summary.csv',
             'NYC': './data/NYC-2016-Summary.csv'}
for city, file_path in data_file.items():
    with open(file_path,'r') as f_in:
        reader = csv.reader(f_in)
        # 去除列名
        trip_lists = list(reader)[1:]
        print('{}市平均骑行时长为：{}分钟'.format(city, round(avg_duration_of_trips(trip_lists), 2)))
        print('{}市骑行时长超过 30 分钟的比例：{:.2%}'.format(city, over_mins_proportion_of_trips(trip_lists,30.00)))


# 测试用例  
# 不要运行
data_file = './examples/BayArea-Y3-Summary.csv'
with open(data_file,'r') as f_in:
    reader = csv.reader(f_in)
    bay_area_list = list(reader)
    bay_area_list = bay_area_list[1:]
print(avg_duration_of_trips(bay_area_list, 'Subscriber'))
print(avg_duration_of_trips(bay_area_list, 'Customer'))


## 使用本框及新框来回答问题 4c。##
## 如果你还没这么做过，你可以考虑修改之前的代码   ##
## 利用一些可重复利用的函数。                            ##
##                                                                     ##
## 小贴士：在海湾示例数据中，你应该发现    ##
## 会员平均骑行时长为 9.5 分钟，散客平均骑行时长则为##
## 54.6 分钟，其它城市区别也这么大吗？     ##
##                                                ##


data_file = {'Washington': './data/Washington-2016-Summary.csv'}
for city, file_path in data_file.items():
    with open(file_path,'r') as f_in:
        reader = csv.reader(f_in)
        # 去除列名
        trip_lists = list(reader)[1:]
        subscriber_avg_duration = avg_duration_of_trips(trip_lists, 'Subscriber')
        customer_avg_duration = avg_duration_of_trips(trip_lists, 'Customer')
        print('{}市会员平均骑行时长为：{}分钟'.format(city, round(subscriber_avg_duration, 2)))
        print('{}市散客平均骑行时长为：{}分钟'.format(city, round(customer_avg_duration, 2)))
        if subscriber_avg_duration-customer_avg_duration > 0:
            print('{}市会员平均骑行时间更长'.format(city))
        else:
            print('{}市散客平均骑行时间更长'.format(city))


# 加载库
import matplotlib.pyplot as plt

# 这个'咒语'能展示图形。
# 内联 notebook，详见：
# http://ipython.readthedocs.io/en/stable/interactive/magics.html
%matplotlib inline 

# 直方图示例，数据来自湾区样本
data = [ 7.65,  8.92,  7.42,  5.50, 16.17,  4.20,  8.98,  9.62, 11.48, 14.33,
        19.02, 21.53,  3.90,  7.97,  2.62,  2.67,  3.08, 14.40, 12.90,  7.83,
        25.12,  8.30,  4.93, 12.43, 10.60,  6.17, 10.88,  4.78, 15.15,  3.53,
         9.43, 13.32, 11.72,  9.85,  5.22, 15.10,  3.95,  3.17,  8.78,  1.88,
         4.55, 12.68, 12.38,  9.78,  7.63,  6.45, 17.38, 11.90, 11.52,  8.63,]
plt.hist(data)
plt.title('Distribution of Trip Durations')
plt.xlabel('Duration (m)')
plt.show()

## 使用本框及新框来收集所有骑行时长并制成列表。##
## 使用 pyplot 函数来为骑行时长生成直方图。 ##
import matplotlib.pyplot as plt

%matplotlib inline 

data_file = {'Washington': './data/Washington-2016-Summary.csv'}
for city, file_path in data_file.items():
    with open(file_path,'r') as f_in:
        reader = csv.reader(f_in)
        lists = list(reader)[1:]
        duration_list = [float(x[0]) for x in lists]
        plt.hist(duration_list, range=(min(duration_list),75))
        plt.title('Distribution of {}\'s Trip Durations'.format(city))
        plt.xlabel('Duration (m)')
        plt.ylabel('Number')
        plt.show()


def draw_duration_plot(lists, title):
        plt.hist(lists,bins=range(0,75,5))
        plt.title(title)
        plt.xlabel('Duration (m)')
        plt.ylabel('Number')
        plt.show()

## 使用本框及新框来回答问题 5##

import matplotlib.pyplot as plt

%matplotlib inline 

data_file = './data/Washington-2016-Summary.csv'
with open(data_file,'r') as f_in:
    reader = csv.reader(f_in)
    lists = list(reader)[1:]
    duration_list = duration_of_trips(lists);
    subscriber_duration_list = duration_of_trips(lists,'Subscriber');
    customer_duration_list = duration_of_trips(lists,'Customer');
   
    draw_duration_plot(subscriber_duration_list,'Distribution of Washington\'s Subscriber Trip Durations')
    draw_duration_plot(customer_duration_list,'Distribution of Washington\'s Customer Trip Durations')

# 不同月份或季度的骑客量有什么区别？哪个月份/季度的骑客量最高？会员骑行量与散客骑行量之比会受月份或季度的影响吗
def monthly_number_of_trips(lists,month=None):
    """
    获取特定月份的骑行数量
    Args:
        lists: list 骑行数据列表
        month: string 月份(数据格式：1-12)
    Return: 
        int 骑客量
    """
    number_list = []
    number_list = list(filter(lambda x:x[1]==month,lists))
    return len(number_list)

def draw_monthly_number_plot(subscriber_lists,customer_lists,title,xlabel,ylabel):
    N = 12
    ind = np.arange(N)    # the x locations for the groups
    width = 0.35       # the width of the bars: can also be len(x) sequence
    
    fig, ax = plt.subplots()
    rects1 = ax.bar(ind, subscriber_lists, width)
    rects2 = ax.bar(ind, customer_lists, width,bottom=subscriber_lists)
    # add some text for labels, title and axes ticks
    ax.set_xlabel(xlabel)
    ax.set_ylabel(ylabel)
    ax.set_title(title)
    ax.set_xticks(ind)
    ax.set_yticks(range(0,10000,1000))
    month_tuple = ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct','Nov', 'Dec')
    ax.set_xticklabels(month_tuple)
    
    # 显示每个bar的值
    i = 0
    for rect in rects1:
        height = subscriber_lists[i] + customer_lists[i]
        i += 1
        ax.text(rect.get_x(), height*1.05, str(height), color='blue')
    ax.legend((rects1[0], rects2[0]), ('Subscriber', 'Customer'), loc='upper left')

    plt.show()

## 使用本框及新框来继续探索数据集。 ##
## 一旦你进行了自己的探索，请写下你的发现 ##
## 请将发现写在上方的 Markdown 框中。 ##
import numpy as np

# 开始计算各月份的骑客量
month_number_list = []
data_file = './data/Washington-2016-Summary.csv'
with open(data_file,'r') as f_in:
    reader = csv.reader(f_in)
    lists = list(reader)[1:]
    # 会员列表
    subscriber_list = filter_trip_user_type(lists, 'Subscriber')
    # 散客列表
    customer_list = filter_trip_user_type(lists, 'Customer')
    subscriber_month_number_list = []
    customer_month_number_list = []
    for month in range(1,13):
        subscriber_month_number_list.append(monthly_number_of_trips(subscriber_list,str(month)))
        customer_month_number_list.append(monthly_number_of_trips(customer_list,str(month)))
    draw_monthly_number_plot(subscriber_month_number_list,customer_month_number_list,'Monthly Distribution of Trips', 'Months', 'Number of Trips')
