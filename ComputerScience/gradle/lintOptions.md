```
android {
    lintOptions {
        // 设置为 true时lint将不报告分析的进度
        quiet true
        // 如果为 true, 则当lint发现错误时停止 gradle构建
        abortOnError false
        // 如果为 true, 则只报告错误
        ignoreWarnings true
        // 如果为 true, 则当有错误时会显示文件的全路径或绝对路径 (默认情况下为true)
        //absolutePaths true
        // 如果为 true, 则检查所有的问题, 包括默认不检查问题
        checkAllWarnings true
        // 如果为 true, 则将所有警告视为错误
        warningsAsErrors true
        // 不检查给定的问题id
        disable 'TypographyFractions','TypographyQuotes'
        // 检查给定的问题 id
        enable 'RtlHardcoded','RtlCompat', 'RtlEnabled'
        // * 仅 * 检查给定的问题 id
        check 'NewApi', 'InlinedApi'
        // 如果为true, 则在错误报告的输出中不包括源代码行
        noLines true
        // 如果为 true, 则对一个错误的问题显示它所在的所有地方, 而不会截短列表, 等等;
        showAll true
        // 重置 lint 配置（使用默认的严重性等设置）;
        lintConfig file("default-lint.xml")
        // 如果为 true, 生成一个问题的纯文本报告（默认为false）
        textReport true
        // 配置写入输出结果的位置；它可以是一个文件或 “stdout”（标准输出）
        textOutput 'stdout'
        // 如果为真, 会生成一个XML报告, 以给Jenkins之类的使用
        xmlReport false
        // 用于写入报告的文件（如果不指定, 默认为lint-results.xml）
        xmlOutput file("lint-report.xml")
        // 如果为真, 会生成一个HTML报告（包括问题的解释, 存在此问题的源码, 等等）
        htmlReport true
        // 写入报告的路径, 它是可选的（默认为构建目录下的 lint-results.html ）
        htmlOutput file("lint-report.html")

         // 设置为 true,  将使所有release 构建都以issus的严重性级别为fatal（severity=false）的设置来运行lint
         // 并且, 如果发现了致命（fatal）的问题, 将会中止构建（由上面提到的 abortOnError 控制）
        checkReleaseBuilds true
        // 设置给定问题的严重级别（severity）为fatal （这意味着他们将会
        // 在release构建的期间检查 （即使 lint 要检查的问题没有包含在代码中)
        //  将指定问题（根据 id 指定）的严重级别（severity）设置为 Fatal  
        fatal 'NewApi', 'InlineApi'
        // 设置给定问题的严重级别为error
        error 'Wakelock', 'TextViewEdits'
        // 设置给定问题的严重级别为warning
        warning 'ResourceAsColor'
        // 设置给定问题的严重级别（severity）为ignore （和不检查这个问题一样）
        ignore 'TypographyQuotes'
    }
}
```