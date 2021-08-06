# Install
    从http://jmeter.apache.org/download_jmeter.cgi 下载 apache-jmeter-5.4.1.zip
    解压后，进到解压目录apache-jmeter-5.4.1/bin下：
    window运行jmeter.bat启动jmeter
    linux运行jmeter.sh启动jmeter
    
    E:\jmeter\apache-jmeter-5.4.1\bin>jmeter -v
        _    ____   _    ____ _   _ _____       _ __  __ _____ _____ _____ ____
       / \  |  _ \ / \  / ___| | | | ____|     | |  \/  | ____|_   _| ____|  _ \
      / _ \ | |_) / _ \| |   | |_| |  _|    _  | | |\/| |  _|   | | |  _| | |_) |
     / ___ \|  __/ ___ \ |___|  _  | |___  | |_| | |  | | |___  | | | |___|  _ <
    /_/   \_\_| /_/   \_\____|_| |_|_____|  \___/|_|  |_|_____| |_| |_____|_| \_\ 5.4.1
    
    Copyright (c) 1999-2021 The Apache Software Foundation
# Usage
    建议使用GUI生成Test Plan，保存未jmx文件后，使用cli进行负载测试
    ================================================================================
    Don't use GUI mode for load testing !, only for Test creation and Test debugging.
    For load testing, use CLI Mode (was NON GUI):
       jmeter -n -t [jmx file] -l [results file] -e -o [Path to web report folder]
    
       -n no GUI mode run test plan
       -t test plan(jmx 文件)
       -l test result file(jtl 文件)
       -j jmeter log file(default在bin下)
       -e generate web test report
       -o report path, empty or not exsit
       
       -H proxy service hostname or ip
       -P proxy service port
       
       jmeter.bat -n -t ..\..\FS_batch_create_delete.jmx -l test.jtl -j chenli.log (测试时在GUI创建时设置了一个)
       
# Solved problem【第一次学习使用，不专业，描述可能还有错误之处】
    1.cli方式运行后，log及jtl当中当中不会记录每个请求的request和response，可以使用JSR223 PostProcessor定制log，语法：
        log.info("Request: " + prev.getSamplerData());
        log.info("Response: " + prev.getResponseDataAsString());
        参考链接：https://www.lfhacks.com/tech/jmeter-save-sampler 
    2.Thread Group下一级的Timer、Assertion会对Group下每个Request有效，比如要循环执行一个Request，sleep的时候一定要在循环控制的
    controller内顶一个Timer
    3.持续判断某个对象变化到某个状态
        通过BeanShell Sampler初始化变量值，vars.put("filesystemStatus","creating")以及vars.put("retry_time","0")
        之后通过While Controller，condition为：
        ${__jexl3(${retry_time} < ${max_retry_count} && "${filesystemStatus}" != "available")}
        来判定是否执行while内语句
    4.Http Request的response code非200时会报错，假设预期有response code非200的请求正确，可以通过JSR223 Assertion获取取样状态，然后
    根据取样状态进行改变的设置
        if("404".equals(SampleResult.getResponseCode())) { // Success
            SampleResult.setSuccessful(true); // Change sampler status to success
        } 
        如上例，http request返回404时，改变smpleer为success，这样jmeter不会报错