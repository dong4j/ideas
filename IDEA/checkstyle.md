# checkstyle 中文配置说明

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
          "-//Puppy Crawl//DTD Check Configuration 1.3//EN"
          "http://checkstyle.sourceforge.net/dtds/configuration_1_3.dtd">

<!--
    Checkstyle configuration that checks the Google coding conventions from Google Java Style
    that can be found at https://google.github.io/styleguide/javaguide.html.

    Checkstyle is very configurable. Be sure to read the documentation at
    http://checkstyle.sf.net (or in your downloaded distribution).

    To completely disable a check, just comment it out or delete it from the file.

    Authors: Max Vetrenko, Ruslan Diachenko, Roman Ivanov.
 -->

<module name = "Checker">
    <property name="charset" value="UTF-8"/>
    <property name="severity" value="warning"/>
    <!-- 检查类型 -->
    <property name="fileExtensions" value="java, properties, xml"/>

    <!-- Checks for whitespace -->
    <!-- 检查是否使用 tab -->
    <module name="FileTabCharacter">
        <!-- 每个文件每行都检查 -->
        <property name="eachLine" value="true"/>
        <message key="containsTab" value="行内含有制表符 tab , 请使用空格代替." />
    </module>

    <!-- 检查文件是否以一个空行结束。 -->
    <module name="NewlineAtEndOfFile">
        <!-- 检查的文件类型 -->
        <property name="fileExtensions" value="java, xml, properties"/>
        <!-- 结尾符号 -->
        <property name="lineSeparator" value="system"/>
        <!-- windws use -->
        <!-- <property name="lineSeparator" value="lf"/> -->
        <message key="noNewlineAtEOF" value="文件末尾有空行, 请删除." />
    </module>

    <!-- 检查两个相同上下文property文件同个属性的键值是否相同 -->
    <module name="Translation" >
        <message key="translation.missingKey" value="关键字 ''{0}'' 未找到,请检查配置文件." />
        <message key="translation.missingTranslationFile" value="找不到配置文件： ''{0}''." />
    </module>

    <!-- 检查property文件内是否有重复的键 -->
    <module name="UniqueProperties">
        <property name="fileExtensions" value="properties" />
        <message key="properties.duplicate.property" value="重复属性： ''{0}'' ({1} 次),请检查配置文件.." />
        <message key="unable.open.cause" value="无法打开： ''{0}'': {1}."/>
    </module>

    <!-- 根据正则表达式检查源文件的标头. -->
    <module name="RegexpHeader"/>

    <!-- 检查 system 打印输出 -->
    <module name="RegexpMultiline">
        <property name="format" value="System\.(out)|(err)\.print(ln)?\("/>
        <property name="message" value="请不要使用 System.out(err).print(ln) 打印,改为 log 输出"/>
    </module>

    <module name="RegexpMultiline">
        <property name="format" value="\r?\n[\t ]*\r?\n[\t ]*\r?\n"/>
        <property name="fileExtensions" value="java,xml,properties"/>
        <property name="message" value="此行下面有多余的空行,请删除"/>
    </module>

    <module name="RegexpSingleline">
        <property name="format" value="^(?!(.*http|import)).{121,}$"/>
        <property name="fileExtensions" value="g, g4"/>
        <property name="message" value="单行字符长度不应大于 120." />
    </module>

    <module name="RegexpOnFilename">
      <property name="folderPattern" value="[\\/]src[\\/]\w+[\\/]java[\\/]"/>
      <property name="fileNamePattern" value="\.java$"/>
      <property name="match" value="false"/>
      <message key="regexp.filepath.mismatch" value="Only java files should be located in the ''src/*/java'' folders."/>
    </module>

    <!-- 文件长度不超过1500行 -->
    <module name="FileLength">
        <property name="max" value="1500"/>
    </module>

    <module name="TreeWalker">

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Annotation(注解) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- see http://checkstyle.sourceforge.net/config_annotation.html -->
        <!--
            AnnotationLocation      检查注释的位置
            AnnotationUseStyle      检查可以控制要使用的注解的样式。
            MissingDeprecated       检查java.lang.Deprecated注解或@deprecated的Javadoc标记是否同时存在。
            MissingOverride         当出现{@inheritDoc}的Javadoc标签时，验证java.lang.Override注解是否出现。
            PackageAnnotation       这项检查可以确保所有包的注解都在package-info.java文件中。
            SuppressWarnings        这项检查允许你指定不允许SuppressWarnings抑制哪些警告信息
            SuppressWarningsHolder
        -->

        <!-- 检查注释的位置 -->
        <module name="AnnotationLocation">
            <property name="id" value="AnnotationLocationMostCases"/>
            <property name="tokens" value="CLASS_DEF, INTERFACE_DEF, ENUM_DEF, METHOD_DEF, CTOR_DEF"/>
        </module>
        <module name="AnnotationLocation">
            <property name="id" value="AnnotationLocationVariables"/>
            <property name="tokens" value="VARIABLE_DEF"/>
            <property name="allowSamelineMultipleAnnotations" value="true"/>
        </module>
        <!-- <module name="AnnotationLocation">
            <property name="allowSamelineMultipleAnnotations" value="false"/>
            <property name="allowSamelineSingleParameterlessAnnotation" value="false"/>
            <property name="allowSamelineParameterizedAnnotation" value="false"/>
            <property name="tokens" value="VARIABLE_DEF, PARAMETER_DEF"/>
        </module> -->

        <!--控制注释的样式-->
        <module name="AnnotationUseStyle">
            <!--注解的参数样式 忽略-->
            <property name="elementStyle" value="ignore"/>
            <!--是否在数组元素后尾随逗号 忽略 -->
            <property name="trailingArrayComma" value="ignore"/>
            <!--检查是否保留结束括号 忽略 -->
            <property name="closingParens" value="ignore"/>
        </module>

        <!-- 检查 @deprecated 注解 -->
        <module name="MissingDeprecated">
            <!-- 使用 @deprecated 时, 必须添加 javadoc 说明 -->
            <property name="skipNoJavadoc" value="true" />
        </module>

        <!-- 检查是否缺失 @Override 注解 -->
        <module name="MissingOverride"/>

        <!-- 确保所有包注释都在package-info.java内 -->
        <module name="PackageAnnotation"/>

        <!-- 检查允许指定SuppressWarnings不允许禁止的警告 -->
        <module name="SuppressWarnings"/>

        <!-- <module name="SuppressWarningsHolder"/> -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Annotation end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Block(块) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- see http://checkstyle.sourceforge.net/config_blocks.html -->
        <!--
            AvoidNestedBlocks   检查不需要的嵌套’{}’。
            EmptyBlock          检查空的代码块。
            EmptyCatchBlock     检查是否出现空白区域。
            LeftCurly           检查’{’和左边的代码块是否在同一行。
            RightCurly          检查’}’。
            NeedBraces          检查是否需要大括号。主要是在if，else时的情况。
        -->

        <!-- 检查是否有嵌套的代码块 -->
        <module name="AvoidNestedBlocks">
            <property name="allowInSwitchCase" value="true" />
            <message key="block.nested" value="避免代码块的嵌套"/>
        </module>

        <!-- 不能出现空白区域 -->
        <module name="EmptyBlock">
            <!-- 代码块中应该包含的类型 -->
            <property name="option" value="TEXT"/>
            <!-- 检查的类型 -->
            <property name="tokens" value="LITERAL_TRY, LITERAL_FINALLY, LITERAL_IF, LITERAL_ELSE, LITERAL_SWITCH"/>
        </module>

        <!-- 检查catch空块以及其中变量注释 -->
        <module name="EmptyCatchBlock">
            <property name="exceptionVariableName" value="expected"/>
        </module>

        <!-- 检查左侧大括号 左侧大括号必须放在前一行代码的行尾 -->
        <module name="LeftCurly">
            <message key="line.previous" value="左侧大括号必须放在前一行代码的行尾，不计入到120个字符内"/>
        </module>

        <!-- 对关键字else、try和catch的右侧大括号放置位置进行检查 -->
        <module name="RightCurly">
            <property name="id" value="RightCurlySame"/>
            <property name="tokens" value="LITERAL_TRY, LITERAL_CATCH, LITERAL_FINALLY, LITERAL_IF, LITERAL_ELSE, LITERAL_DO"/>
        </module>
        <module name="RightCurly">
            <property name="id" value="RightCurlyAlone"/>
            <property name="option" value="alone"/>
            <property name="tokens" value="CLASS_DEF, METHOD_DEF, CTOR_DEF, LITERAL_FOR, LITERAL_WHILE, STATIC_INIT, INSTANCE_INIT"/>
        </module>

        <!-- 检查只有必须有{},确省为必须,主要在if,else时有这样的情况 -->
        <module name="NeedBraces">
            <message key="needBraces" value="''{0}'' 结构必须要用大括号 '''{}'''s."/>
        </module>
        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Block end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Class Design(类设计) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- see http://checkstyle.sourceforge.net/config_design.html @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!--
            DesignForExtension              检查类是否为扩展设计。
            FinalClass                      检查只有private构造函数的类是否声明为final。
            HideUtilityClassConstructor     检查工具类是否有putblic的构造器。
            InnerTypeLast                   检查嵌套类的声明是否在方法与字段声明后面
            InterfaceIsType                 检查接口是否仅定义类型。
            MutableException                确保异常是不可变的。
            OneTopLevelClass                检查顶级类的接口或枚举是否位于自己的源文件中
            ThrowsCount                     限制抛出异常的数量。
            VisibilityModifier              检查类成员的可见度。
        -->

        <!-- 要求一个方法必须声明为Extension的,否则必须声明为abstract, final or empty -->
        <module name="DesignForExtension"/>

        <!-- 检查private构造是否声明为final,这里有个问题,在Java中构造是不能声明为final的 -->
        <!-- <module name="FinalClass"/> -->

        <!-- 只定义了静态方法的类不应该定义一个公有的构造器 (类应定义一个构造器/工具类应隐藏 public 构造器)-->
        <module name="HideUtilityClassConstructor">
	    <message key="hide.utility.class" value="工具类应隐藏 public 构造器"/>
	</module>

        <!-- 检查嵌套类的声明是否在方法与字段声明后面 -->
        <module name="InnerTypeLast"/>

        <!-- 检查接口是否只定义了变量而没有定义方法，因为接口应该用来描述一个类型，所以只定义变量而不定义方法是不恰当的 -->
        <module name="InterfaceIsType">
            <!-- 是否允许定义空接口 -->
            <property name="allowMarkerInterfaces" value="true" />
        </module>

        <!-- 确保异常是不可变的 -->
        <module name="MutableException"/>

        <!-- 检查顶级类的接口或枚举是否位于自己的源文件中 -->
        <module name="OneTopLevelClass"/>

        <!-- 异常抛出数量定义 -->
        <!-- <module name="ThrowsCount">
            <property name="max" value="3"/>
        </module> -->

        <!-- 检查变量的可见性,缺省只允许static final 为public,否则只能为private -->
        <!-- <module name="VisibilityModifier">
            <property name="packageAllowed" value="true"/>
            <property name="protectedAllowed" value="true"/>
        </module> -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Class Design end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Coding(代码) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- see http://checkstyle.sourceforge.net/config_coding.html -->
        <!--
            ArrayTrailingComma                  检查数组初始化是否以逗号结束。
            AvoidInlineConditionals             检查inline的条件操作。
            CovariantEquals                     检查类是否覆盖了equals(java.lang.Object)。
            DeclarationOrder                    检查类和接口中的声明顺序。
            DefaultComesLast                    检查default的clause是否在switch代码段的最后。
            EmptyStatement                      检查空的代码段。
            EqualsAvoidNull                     检查一个可能为null的字符串是否在equals()比较的左边。
            EqualsHashCode                      检查类是否覆盖了equals()和hashCode()。
            ExplicitInitialization              检查类和对象成员是否初始化为默认值。
            FallThrough                         检查switch代码的case中是否缺少break，return，throw和continue。
            FinalLocalVariable                  检查未改变过的局部变量是否声明为final。
            HiddenField                         检查局部变量或参数是否隐藏了类中的变量。
            IllegalCatch                        检查是否catch了不能接受的错误。
            IllegalInstantiation                检查是否使用工厂方法实例化。
            IllegalThrows                       检查是否抛出了未声明的异常。
            IllegalToken                        检查非法的分隔符。
            IllegalTokenText                    检查非法的分隔符的下个字符。
            IllegalType                         检查未使用过的类。
            InnerAssignment                     检查子表达式中是否有赋值操作。
            MagicNumber                         检查是否有“magic numbers”。
            MissingCtIor                        检查类（除了抽象的）定义一个构造函数，不要依赖默认。
            MissingSwitchDefault                检查switch语句是否有default的clause。
            ModifiedControlVariable             检查循环控制的变量是否在代码块中被修改。
            MultipleStringLiterals              检查一个文件中是否有多次出现的字符串。
            MultipleVariableDeclarations        检查代码段和代码行中是否有多次变量声明。
            NestedForDepth                      限制块嵌套为指定深度以内
            NestedIfDepth                       检查嵌套的层次深度。
            NestedTryDepth                      检查try的层次深度。
            NoClone                             检查是否覆盖了clone()。
            NoFinalizer                         检查是否有定义finalize()。
            OneStatementPerLine
            OverloadMethodsDeclarationOrder     检查Overload的顺序。
            PackageDeclaration                  检查类中是否有声明package。
            ParameterAssignment                 检查不允许的参数赋值。
            RequireThis                         检查代码中是否有 "this."。
            ReturnCount                         限制return代码段的数量。
            SimplifyBooleanExpression           检查是否有过度复杂的布尔表达式。
            SimplifyBooleanReturn               检查是否有过于复杂的布尔返回代码段。
            StringLiteralEquality               检查字符串是否有用 == 或 != 进行操作。
            SuperClone                          检查覆盖的clone()是否有调用super.clone()。
            SuperFinalize                       检查覆盖的finalize()是否有调用super.finalize()。
            UnnecessaryParentheses              检查是否有使用不需要的圆括号。
            VariableDeclarationUsageDistance    检查声明变量与其第一次用的距离
        -->

        <!-- 检查初始化数祖时，最后一个元素后面是否加了逗号，如果左右大括号都在同一行，则可以不加逗号 -->
        <module name="ArrayTrailingComma"/>

        <!-- 检查是否在同一行初始化 -->
        <module name="AvoidInlineConditionals">
            <message key="inline.conditional.avoid" value="避免内部条件语句，不易于代码阅读."/>
        </module>

        <!-- 检查类是否覆盖了equals -->
        <module name="CovariantEquals"/>

        <!-- class 或 interface 中的顺序如下：
            1. class声明: public > protected > packagelevel > private
            2. 变量声明: public > protected > packagelevel > private
            3. 构造函数
            4. 方法  -->
        <module name="DeclarationOrder">
	    <message key="declaration.order.access" value="属性访问器定义顺序错误."/>
            <message key="declaration.order.constructor" value="构造器定义顺序错误."/>
	    <message key="declaration.order.instance" value="实例属性定义顺序错误."/>
	    <message key="declaration.order.static" value="静态属性定义顺序错误."/>
	</module>

        <!-- 检查default的clause是否在switch代码段的最后。 -->
        <module name="DefaultComesLast"/>

        <!-- 不许出现空语句 -->
        <module name="EmptyStatement" >
            <message key="empty.statement" value="空代码块."/>
        </module>

        <!--避免 null.equals("sss")情况-->
        <module name="EqualsAvoidNull">
	    <message key="equals.avoid.null" value="字符串常量应出现在 equals 比较的左侧."/>
	</module>
        <!-- 每个类都实现了equals()和hashCode() -->
        <module name="EqualsHashCode" />
        <!-- 确保某个class 在被使用时都已经被初始化成默认值(对象是null,数字和字符是0,boolean 变量是false.) -->
        <module name="ExplicitInitialization" />

        <!-- 检查switch代码的case中是否缺少break，return，throw和continue。 -->
        <module name="FallThrough"/>

        <!-- 检查未改变过的局部变量是否声明为final。 -->
        <module name="FinalLocalVariable">
            <message key="final.variable" value="变量 ''{0}'' 未被改变过,应被声明为 final 的." />
        </module>

        <!-- 检查局部变量或参数是否隐藏了类中的变量。 -->
        <module name="HiddenField">
            <!-- 忽略构造器覆盖 -->
            <property name="ignoreConstructorParameter" value="true"/>
            <!-- 忽略set覆盖 -->
            <property name="ignoreSetter" value="true"/>
        </module>

        <!-- 不能catch java.lang.Exception -->
        <module name="IllegalCatch">
            <property name="illegalClassNames" value="java.lang.Exception"/>
            <message key="illegal.catch" value="Catching ''{0}'' 是不允许的."/>
        </module>

        <!-- 检查是否使用工厂方法实例化。 -->
        <!-- <module name="IllegalInstantiation"/> -->

        <!-- 检查是否抛出了未声明的异常。 -->
        <module name="IllegalThrows"/>

        <!-- 不许使用switch,"a++"这样可读性很差的代码 -->
        <module name="IllegalToken" />

        <!-- 检查非法的分隔符 -->
        <module name="IllegalTokenText">
            <!-- tokens to check -->
            <property name="tokens" value="STRING_LITERAL, CHAR_LITERAL"/>
            <!-- Controls whether to ignore case when matching. default = false -->
            <property name="ignoreCase" value="false"/>
            <!-- illegal pattern -->
            <property name="format" value="\\u00(09|0(a|A)|0(c|C)|0(d|D)|22|27|5(C|c))|\\(0(10|11|12|14|15|42|47)|134)"/>
            <!-- Message which is used to notify about violations; if empty then the default message is used. -->
            <property name="message" value="Consider using special escape sequence instead of octal value or Unicode escaped value."/>
        </module>

        <!-- 检查未使用过的类。 -->
        <module name="IllegalType"/>
        <!-- 不许内部赋值 -->
        <module name="InnerAssignment"/>

        <!-- 不允许魔法数 -->
        <module name="MagicNumber">
            <property name="tokens" value="NUM_DOUBLE, NUM_INT" />
            <!-- 忽略hashcode方法中的数字 -->
            <property name="ignoreHashCodeMethod" value="true"/>
            <message key="magic.number" value="''{0}'' 是一个魔法数(即常数)."/>
        </module>

        <!-- 检查类（除了抽象的）定义一个构造函数，不要依赖默认。 -->
        <module name="MissingCtor">
            <message key="missing.ctor" value="请为当前类显式定义一个无参构造方法, 避免不必要的错误." />
        </module>

        <!-- 检查switch语句是否有default的clause。 -->
        <module name="MissingSwitchDefault"/>

        <!-- 循环控制变量不能被修改 -->
        <module name="ModifiedControlVariable" />
        <!-- 不许有同样内容的String -->
        <module name="MultipleStringLiterals">
            <property name="allowedDuplicates" value="2"/>
            <message key="multiple.string.literal" value="字符串: {0} 在本文件中出现了 {1} 次,请定义常量并替换." />
        </module>

        <!-- 检查代码段和代码行中是否有多次变量声明。 -->
        <module name="MultipleVariableDeclarations">
            <message key="multiple.variable.declarations" value="每一行只能定义一个变量."/>
            <message key="multiple.variable.declarations.comma" value="每一个变量的定义必须在它的声明处,且在同一行."/>
        </module>

        <!-- 限制for循环最多嵌套3层 -->
        <module name="NestedForDepth">
            <property name="max" value="3"/>
        </module>
        <!-- if最多嵌套3层 -->
        <module name="NestedIfDepth">
            <property name="max" value="3"/>
            <message key="nested.if.depth" value="if-else嵌套语句个数为 {0,number,integer} (最大允许嵌套语句个数为: {1,number,integer})."/>
        </module>
        <!--try catch 异常处理数量 3-->
        <module name="NestedTryDepth ">
            <property name="max" value="3"/>
        </module>

        <!-- 检查是否覆盖了clone()。 -->
        <module name="NoClone"/>

        <!-- 检查是否有定义finalize -->
        <module name="NoFinalizer"/>

        <module name="OneStatementPerLine"/>

        <!-- 检查Overload的顺序。 -->
        <module name="OverloadMethodsDeclarationOrder"/>

        <!-- 确保一个类有package声明 -->
        <module name="PackageDeclaration" >
            <message key="missing.package.declaration" value="缺少包的定义."/>
            <message key="package.dir.mismatch" value="包定义与目录名不匹配 ''{0}''."/>
        </module>
        <!-- 不许对方法的参数赋值 -->
        <module name="ParameterAssignment" >
            <message key="parameter.assignment" value="参数赋值 ''{0}'' 是不允许的."/>
        </module>

        <!-- 检查代码中是否有 "this."。
            同一个类中调用非静态方法时显式使用 this. 调用
        -->
        <module name="RequireThis">
            <property name="checkFields" value="false"/>
            <property name="checkMethods" value="false"/>
        </module>

        <!-- 一个方法中最多有3个return -->
        <module name="ReturnCount">
            <property name="max" value="3" />
            <property name="format" value="^$" />
            <message key="return.count" value="Return 个数 {0,number,integer} (最大允许个数为: {1,number,integer})."/>
        </module>

        <!-- 检查是否有过度复杂的布尔表达式。 -->
        <module name="SimplifyBooleanExpression" />
        <!-- 检查是否有过于复杂的布尔返回代码段。 -->
        <module name="SimplifyBooleanReturn" />
        <!-- String的比较不能用!= 和 == -->
        <module name="StringLiteralEquality" >
            <message key="string.literal.equality" value="字符串比较必须使用 equals(), 而不是 ''{0}''."/>
        </module>
        <!-- clone方法必须调用了super.clone() -->
        <module name="SuperClone" >
            <message key="missing.super.call" value="方法 ''{0}'' 需要调用 ''super.{0}''."/>
        </module>
        <!-- finalize 必须调用了super.finalize() -->
        <module name="SuperFinalize" >
            <message key="missing.super.call" value="方法 ''{0}'' 需要调用 ''super.{0}''."/>
        </module>
        <!-- 不必要的圆括号 -->
        <module name="UnnecessaryParentheses" />

        <!-- 检查声明变量与其第一次用的距离 -->
        <module name="VariableDeclarationUsageDistance">
            <property name="allowedDistance" value="4"/>
            <property name="ignoreVariablePattern" value="^temp.*"/>
            <property name="validateBetweenScopes" value="true"/>
            <property name="ignoreFinal" value="true"/>
            <!-- 变量''{0}''声明及第一次使用距离{1}行（最多：{2} 行） -->
            <!-- 变量''{0}''声明及第一次使用距离{1}行（最多：{2} 行）。若需要存储该变量的值，请将其声明为final的（方法调用前声明以避免副作用影响原值） -->
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Coding end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Headers start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sf.net/config_header.html -->
        <!--
            Header          检查文件是否以指定文件开始
            RegexpHeader    根据正则表达式检查源文件的标头
         -->

        <!-- 检查文件是否以指定文件开始,这里最好是放一些版权信息和工程描述 -->
        <module name="Header">
            <!-- headerFile:指定的文件 -->
            <property name="headerFile" value="java.header"/>
            <!-- ignoreLines:忽略哪些行,以","分隔 -->
            <property name="ignoreLines" value="2, 3, 4, 5"/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Headers end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for imports(导入) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_imports.html -->
        <!--
            AvoidStarImport     检查是否有使用*.*进行import。
            AvoidStaticImport   检查是否有静态import。比如是否导入了java.lang包中的内容。
            CustomImportOrder   检查import包的分组和顺序
            IllegalImport       检查是否import了违法的包。默认拒绝import所有sun.*包。
            ImportControl       控制可import的包。在一个较大的project可限制使用过多的第三方包
            ImportOrder         检查import的分组和顺序。
            RedundantImport     检查是否有重复的import。
            UnusedImports       检查是否有未使用的import。
         -->

        <!-- 限制导入.*包的检查 -->
        <module name="AvoidStarImport">
            <!-- 允许导入java.io.*,java.net.*,java.lang.Math.*,其它不允许 -->
            <property name="excludes" value="java.io,java.net,java.lang.Math"/>
            <!-- 实例；import java.util.*;.-->
            <property name="allowClassImports" value="false"/>
            <!-- 允许静态导入 实例 :import static org.junit.Assert.*;-->
            <property name="allowStaticMemberImports" value="true"/>
        </module>

        <!-- 检查是否含有静态导入 静态导入可能会导致代码可读性差，因为它可能不再清楚某个成员所在的类-->
        <module name="AvoidStaticImport">
           <property name="excludes" value="java.lang.System.out,java.lang.Math.*"/>
        </module>

        <!-- 检查import包的分组和顺序 -->
        <module name="CustomImportOrder">
            <property name="sortImportsInGroupAlphabetically" value="false"/>
            <property name="separateLineBetweenGroups" value="true"/>
            <!-- 分组规则 -->
            <!-- <property name="customImportOrderRules" value="STATIC###THIRD_PARTY_PACKAGE"/> -->
            <property name="customImportOrderRules" value="STATIC###STANDARD_JAVA_PACKAGE###SPECIAL_IMPORTS###THIRD_PARTY_PACKAGE"/>
            <property name="specialImportsRegExp" value="^org\."/>
            <property name="thirdPartyPackageRegExp" value="^com\."/>
        </module>

        <!-- 检查是否import了非法包, 默认全部的 sun.* , 不保证工作在所有兼容java的平台-->
        <module name="IllegalImport">
            <!-- 自定义非法的包 -->
            <property name="illegalPkgs" value="java.io, java.sql"/>
            <!-- 自定义非法的class -->
            <property name="illegalClasses" value="java.util.Date, java.sql.Connection"/>
            <!-- 允许正则表达式匹配 -->
            <property name="regexp" value="true"/>
        </module>

        <!-- 控制可import的包。在一个较大的**project**可限制使用过多的第三方包 -->
        <module name="ImportControl">
            <property name="file" value="config/import-control.xml"/>
            <property name="path" value="^.*[\\/]src[\\/]main[\\/].*$"/>
        </module>

        <!-- 导入排序 -->
        <module name="ImportOrder">
            <!-- groups:分组,哪些是一组的 -->
            <property name="groups" value="java,javax"/>
            <!-- ordered:同一个组内是否排序,true排序,确省为true -->
            <property name="ordered" value="true"/>
            <!-- separated:各个组之间是否需要用空行分隔,确省为false -->
            <property name="separated" value="true"/>
            <!-- caseSensitive:是否是大小写敏感的,确省是 true -->
            <property name="caseSensitive" value="true"/>
        </module>

         <!-- 限制导入多余的包,例如java.lang.String -->
        <module name="RedundantImport"/>

        <!-- 没用的import检查，比如：1.没有被用到2.重复的3.import java.lang的4.import 与该类在同一个package的 -->
        <module name="UnusedImports" >
            <message key="import.unused" value="没被使用过 import - {0}."/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for imports end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Javadoc Comments(java注释) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_javadoc.html -->
        <!--
            AtclauseOrder                       检查java-doc块标签或者标签顺序
            JavadocMethod                       检查javadoc的方法或构造函数
            JavadocPackage                      检查所有软件包是否具有相应文档
            JavadocParagraph                    检查javadoc的段落
            JavadocStyle                        自定义检查以验证javadoc
            JavadocTagContinuationIndentation
            JavadocType                         检查javadoc的类型
            JavadocVariable                     检查变量是否具有javadoc注释
            NonEmptyAtclauseDescription         检查 @param, @return, @throws, @deprecated
            SingleLineJavadoc                   检查javadoc摘要句是否包含不推荐的短语
            SummaryJavadoc                      检查javadoc摘要句是否包含不推荐的短语
            WriteTag                            输出javadoc标签作为信息
        -->

        <!-- 检查java-doc块标签或者标签顺序 -->
        <module name="AtclauseOrder">
            <property name="tagOrder" value="@param, @return, @throws, @deprecated"/>
            <property name="target" value="CLASS_DEF, ENUM_DEF, METHOD_DEF, CTOR_DEF, VARIABLE_DEF"/>
        </module>

        <!-- 检查javadoc的方法或构造函数 -->
        <module name="JavadocMethod">
            <!-- 检查范围 -->
            <property name="scope" value="public"/>
            <!-- 是否忽略对参数注释的检查 -->
            <property name="allowMissingParamTags" value="true"/>
            <!--  是否忽略对throws注释的检查 -->
            <property name="allowMissingThrowsTags" value="true"/>
            <!-- 是否忽略对return注释的检查 -->
            <property name="allowMissingReturnTag" value="true"/>
            <property name="minLineCount" value="2"/>
            <property name="allowedAnnotations" value="Override, Test"/>
            <property name="allowThrowsTagsForSubclasses" value="true"/>
            <!--允许get set 方法没有注释-->
            <property name="allowMissingPropertyJavadoc" value="true"/>
            <!-- 可以不声明RuntimeException -->
            <property name="allowUndeclaredRTE" value="true"/>
            <message key="javadoc.missing" value="方法注释：缺少Javadoc注释。"/>
        </module>

        <!-- 检查所有软件包是否具有相应文档 package-info.java-->
        <!-- <module name="JavadocPackage" /> -->

        <!-- 检查javadoc的段落 -->
        <module name="JavadocParagraph" />

        <!-- 检查Javadoc的格式 -->
        <module name="JavadocStyle">
            <property name="scope" value="public"/>
            <!-- Comment的第一句的末尾是否要有一个句号,true必须有,default为true -->
            <property name="checkFirstSentence" value="false"/>
            <!-- 检查空的javadoc -->
            <property name="checkEmptyJavadoc" value="false"/>
            <!-- 检查错误的HTML脚本,比如不匹配,true检查,default为true -->
            <property name="checkHtml" value="false"/>
        </module>

        <!-- 不知道是个啥?????? -->
        <module name="JavadocTagContinuationIndentation">
            <property name="offset" value="4"/>
        </module>

        <!-- 检查javadoc的类型 -->
        <module name="JavadocType">
            <!-- 检查的类的范围 -->
            <property name="scope" value="private"/>
            <!-- 排除范围 -->
            <!-- <property name="excludeScope" value="public"/> -->
            <!-- 检查author标签的格式 -->
            <property name="authorFormat" value="\S"/>
            <!-- 检查version标签的格式 -->
            <property name="versionFormat" value="\$Revision.*\$"/>
            <!-- 不检查参数 -->
            <property name="allowMissingParamTags" value="true"/>
            <!-- 不检查未知标记 -->
            <property name="allowUnknownTags" value="true"/>
            <property name="tokens" value="CLASS_DEF,INTERFACE_DEF"/>
        </module>

        <!-- 检查某个变量的javadoc -->
        <module name="JavadocVariable">
            <property name="scope" value="public"/>
            <!-- 排除 protected 的注释检查 -->
            <property name="excludeScope" value="protected"/>
            <property name="ignoreNamePattern" value="log|logger"/>
            <message key="javadoc.missing" value="变量注释：缺少Javadoc注释。"/>
        </module>

        <!-- Default configuration that will check @param, @return, @throws, @deprecated -->
        <module name="NonEmptyAtclauseDescription"/>

        <!-- 检查javadoc块是否可以适应单行，并且不包含at-clause -->
        <module name="SingleLineJavadoc">
            <property name="ignoreInlineTags" value="false"/>
        </module>

        <!-- 检查javadoc摘要句是否包含不推荐的短语 -->
        <module name="SummaryJavadoc">
            <property name="forbiddenSummaryFragments" value="^@return the *|^This method returns |^A [{]@code [a-zA-Z0-9]+[}]( is a )"/>
        </module>

        <!-- 输出javadoc标签作为信息 -->
        <module name="WriteTag">
            <property name="tag" value="@incomplete"/>
            <property name="tagFormat" value="\S"/>
            <property name="severity" value="ignore"/>
            <property name="tagSeverity" value="warning"/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Javadoc Comments end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->


        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Metrics(复杂度) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_metrics.html -->
        <!--
            BooleanExpressionComplexity     布尔表达式复杂度 限制一个表达式中的&&、||、&、|、^等逻辑运算符的数量。
            ClassDataAbstractionCoupling    类的数据抽象耦合 测量给定类中的其他类的实例化操作的次数。
                                                这种类型的耦合并不是由于继承或者面向对象范型而产生的。
                                                一般而言，任何将其他抽象数据类型作为成员的抽象数据类型都具有数据抽象耦合；
                                                因此，如果一个类中的某个局部变量是另一个类的实例（对象），那么就存在数据抽象耦合（DAC）。
                                                DAC越高，系统的数据结构（类）就会越复杂。
            ClassFanOutComplexity           类的扇出复杂度 一个给定类所依赖的其他类的数量。
                                                这个数量的平方还可以用于表示函数式程序（基于文件）中需要维护总量的最小值。
            CyclomaticComplexity            循环复杂度 检查循环复杂度是否超出了指定的限值。
                                                该复杂度由构造器、方法、静态初始化程序、实例初始化程序中的
                                                if、while、do、for、?:、catch、switch、case等语句，以及&&和||运算符的数量所测量。
                                                它是遍历代码的可能路径的一个最小数量测量，因此也是需要的测试用例的数量。
                                                通常1-4是很好的结果，5-7较好，8-10就需要考虑重构代码了，
                                                如果大于11，则需要马上重构代码！
            JavaNCSS                        非注释源码语句 通过对非注释源码语句（NCSS）进行计数，确定方法、类、文件的复杂度。
                                                这项检查遵守Chr. Clemens Lee编写的JavaNCSS-Tool中的规范。
                                                NCSS度量就是不包含注释的源代码行数，（近似）等价于分号和左花括号的计数。
                                                一个类的NCSS就是它所有方法的NCSS、它的内部类的NCSS、成员变量声明数量的总和。
                                                一个文件的NCSS就是它所包含的所有顶层类的NCSS、imports语句和包声明语句数量的总和。
                                                太大的方法和类会难以阅读，并且维护成本会很高。
                                                一个较大的NCSS数值通常意味着对应的方法或类承担了过多的责任和/或功能，应当分解成多个较小的单元。
            NPathComplexity                 NPATH度量会计算遍历一个函数时，所有可能的执行路径的数量。
                                                它会考虑嵌套的条件语句，以及由多部分组成的布尔表达式（例如，A && B，C || D，等等）。
                                                在Nejmeh的团队中，每个单独的例程都有一个取值为200的非正式的NPATH限值；
                                                超过这个限值的函数可能会进行进一步的分解，或者至少一探究竟。
        -->

        <!-- 限制布尔运算符的复杂度（&& || 等） -->
        <module name="BooleanExpressionComplexity">
            <!-- 不超过 3 -->
            <property name="max" value="3"/>
            <property name="tokens" value="LAND, BAND, LOR, BOR, BXOR"/>
        </module>

        <!-- 类数据的抽象耦合，不超过 7 -->
        <module name="ClassDataAbstractionCoupling" />

        <!-- 类的分散复杂度，不超过 20 -->
        <module name="ClassFanOutComplexity" />

        <!-- 函数的分支复杂度，不超过 10 -->
        <module name="CyclomaticComplexity" />

        <module name="JavaNCSS">
            <property name="methodMaximum" value="40"/>
        </module>

        <!-- NPath复杂度，不超过 200 -->
        <module name="NPathComplexity" />

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Metrics end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Miscellaneous(杂项) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_misc.html -->
        <!--
            ArrayTypeStyle                  检查数组类型定义的样式。
            AvoidEscapedUnicodeCharacters   限制使用Unicode escapes
            CommentsIndentation             控制注释和代码之间的缩进
            DescendantToken
            FileContentsHolder
            FinalParameters                 检查方法名、构造函数、catch块的参数是否是final的。
            Indentation                     检查代码中正确的缩进。
            NewlineAtEndOfFile              检查文件是否以一个空行结束。
            OuterTypeFilename               检查类名和文件名是否匹配。
            TodoComment                     检查TODO:注释。
            TrailingComment                 行尾注释
            Translation                     检查property文件中是否有相同的key-value。
            UncommentedMain                 检查是否有未注释的main方法。
            UniqueProperties                检查property文件内是否有重复的键
            UpperEll                        检查long型约束是否有大写的“L”。
        -->

        <!-- 检查数组类型的定义是String[] args，而不是String args[] -->
        <module name="ArrayTypeStyle">
            <message key="array.type.style" value="数组中括号位置不对,建议如:String[] args."/>
        </module>    

        <!-- 限制使用Unicode escapes -->
        <module name="AvoidEscapedUnicodeCharacters">
            <!-- Allow use escapes for non-printable(control) characters. default = false -->
            <property name="allowEscapesForControlCharacters" value="true"/>
            <!-- Allow use escapes if trail comment is present. default = false -->
            <property name="allowByTailComment" value="true"/>
            <!-- Allow if all characters in literal are escaped. default = false -->
            <property name="allowNonPrintableEscapes" value="true"/>
            <!-- Allow non-printable escapes. default = false -->
            <property name="allowIfAllCharactersEscaped" value="false"/>
        </module>

        <!-- 控制注释和代码之间的缩进 -->
        <module name="CommentsIndentation"/>

        <!-- <module name="DescendantToken"/> -->

        <!-- <module name="FileContentsHolder"/> -->

        <!-- 确保方法、构造函数函数、循环等内参数为final -->
        <!-- <module name="FinalParameters"/> -->

        <!-- 检查java代码的缩进 默认配置：，：0。 4个空格 -->
        <module name="Indentation">
            <!-- 基本缩进 4个空格 -->
            <property name="basicOffset" value="4"/>
            <!-- 新行的大括号 -->
            <property name="braceAdjustment" value="0"/>
            <!-- 新行的空格数 -->
            <property name="caseIndent" value="4"/>
            <!-- throws子句 空格数 -->
            <property name="throwsIndent" value="2"/>
            <!-- 换行后的空格数 -->
            <property name="lineWrappingIndentation" value="4"/>
            <!-- 数组初始化时的空格数 -->
            <property name="arrayInitIndent" value="4"/>
            <!-- 换行时是否按照 arrayInitIndent 检查-->
            <property name="forceStrictCondition" value="false"/>
        </module>

        <!-- 检查类名和文件名是否匹配。例如，Foo类必须有一个文件名为foo.java -->
        <module name="OuterTypeFilename"/>

        <!-- TODO的检查,表示不要出现todo未办事项目-->
        <module name="TodoComment">
            <property name="format" value="TODO\W+"/>
        </module>

        <!-- 如果有 //, 必须写注释内容 -->
        <module name="TrailingComment">
            <property name="format" value="^\\s*$"/>
        </module>

        <!-- 检查是否有没有被注掉或者删除的main方法 -->
        <module name="UncommentedMain">
            <!-- 设置排除的文件 -->
            <!-- <property name="excludedClasses" value=".*Main$"/> -->
        </module>

        <!-- 当定义一个常量时,希望使用大写的L来代替小写的l,原因是小写的l和数字1很象 -->
        <module name="UpperEll">
            <message key="upperEll" value="必须使用大写字母 ''L''."/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Miscellaneous end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Modifiers(关键字) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_modifier.html -->
        <!--
            ModifierOrder       检查修饰符的顺序
                                    每个关键字都有正确的出现顺序。比如 public static final XXX 是对一个常量的声明。
                                    如果使用 static public final 就是错误的
            RedundantModifier   检查接口和annotation中是否有重复的修饰符。
        -->
        <module name="ModifierOrder" >
            <message key="mod.order" value="''{0}'' 修饰符没有按照 JLS 的建议顺序."/>
            <message key="annotation.order" value="''{0}'' 注释修饰符不能在非注释修饰符前面."/>
        </module>
        <!-- 多余的关键字 -->
        <module name="RedundantModifier" >
            <message key="redundantModifier" value="冗余 ''{0}'' 修饰符."/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Modifiers end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Naming Conventions(命名规则) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_naming.html -->
        <!--
            AbbreviationAsWordInName    驼峰命名法检查
            AbstractClassName           检查抽象类的命名规则
            CatchParameterName          确认参数名是否符合规定的格式
            ClassTypeParameterName      确保类的参数名是否符合所定格式
            ConstantName                常量名的检查
            InterfaceTypeParameterName  检查接口类型参数名称是否符合格式
            LocalFinalVariableName      检查局部常量的命名是否符合格式
            LocalVariableName           检查局部变量的命名是否符合格式
            MemberName                  检查类里变量名是否符合格式
            MethodName                  检查方法命名是否符合格式
            MethodTypeParameterName     检查方法的参数名是否符合格式
            PackageName                 检查包名是否符合格式
            ParameterName               检查参数名是否符合格式
            StaticVariableName          检查静态变量名是否符合格式
            TypeName                    检查类名和接口名是否符合格式
        -->

        <!-- 驼峰命名法检查 -->
        <module name="AbbreviationAsWordInName">
            <!-- 忽略 final 属性命名检查 -->
            <property name="ignoreFinal" value="true"/>
            <!-- 忽略 static 属性命名检查 -->
            <property name="ignoreStatic" value="true"/>
            <!-- 允许连续大写的数量 -->
            <property name="allowedAbbreviationLength" value="2"/>
            <!-- 允许不使用驼峰命名法检查的文件类型 -->
            <property name="allowedAbbreviations" value="XML,URL"/>
            <!-- 忽略重写的方法名检查 -->
            <property name="ignoreOverriddenMethods" value="true"/>
            <!-- 检查类型 -->
            <property name="tokens" value="CLASS_DEF, INTERFACE_DEF, ENUM_DEF, ANNOTATION_DEF, ANNOTATION_FIELD_DEF, PARAMETER_DEF, VARIABLE_DEF, METHOD_DEF"/>
        </module>

        <!-- 确省必须以Abstract开始或者以Factory结束 -->
        <module name="AbstractClassName">
            <property name="format" value="^Abstract.*$|^.*Factory$"/>
        </module>

        <!-- 确认参数名是否符合规定的格式 -->
        <module name="CatchParameterName">
            <property name="format" value="^[a-z]([a-z0-9][a-zA-Z0-9]*)?$"/>
            <message key="name.invalidPattern"
             value="Catch parameter name ''{0}'' must match pattern ''{1}''."/>
        </module>

        <!-- 确保类的参数名是否符合所定格式 -->
        <module name="ClassTypeParameterName">
            <property name="format" value="(^[A-Z][0-9]?)$|([A-Z][a-zA-Z0-9]*[T]$)"/>
            <message key="name.invalidPattern"
             value="Class type name ''{0}'' must match pattern ''{1}''."/>
        </module>

        <!-- 常量名的检查 -->
        <module name="ConstantName">
            <property name="format" value="(^[A-Z0-9_]{0,25}$)"/>  
            <!-- <message key="name.invalidPattern" value="ConstantName ''{0}'' must match pattern ''{1}''."/> -->
            <message key="name.invalidPattern" value="常量名 ''{0}'' 必须 全部大写, 单词之间使用 _ 分隔,最大长度25."/>
        </module>

        <!-- 检查接口类型参数名称是否符合格式 -->
        <module name="InterfaceTypeParameterName">
            <property name="format" value="(^[A-Z][0-9]?)$|([A-Z][a-zA-Z0-9]*[T]$)"/>
            <message key="name.invalidPattern"
             value="Interface type name ''{0}'' must match pattern ''{1}''."/>
        </module>

        <!-- 检查局部常量的命名是否符合格式 -->
        <module name="LocalFinalVariableName">
            <message key="name.invalidPattern"
                value="局部常量名 ''{0}'' must match pattern ''{1}''."/>
        </module>

        <!-- 检查局部变量的命名是否符合格式 -->
        <module name="LocalVariableName">
            <property name="tokens" value="VARIABLE_DEF"/>
            <property name="format" value="^[a-z]([a-z0-9][a-zA-Z0-9]*)?$"/>
            <message key="name.invalidPattern"
             value="局部变量名 ''{0}'' must match pattern ''{1}''."/>
        </module>

        <!-- 检查类里变量名是否符合格式 -->
        <module name="MemberName">
            <property name="format" value="(^[a-z][a-z0-9][a-zA-Z0-9]{0,19}$)"/> 
            <message key="name.invalidPattern"
             value="成员变量名 ''{0}'' 必须以 首字母小写,接下来的单词都以大写字母开头,最大长度19."/>
        </module>

        <!-- 检查方法命名是否符合格式 -->
        <module name="MethodName">
            <property name="format" value="^[a-z][a-z0-9][a-zA-Z0-9_]*$"/>
            <message key="name.invalidPattern" value="方法名 ''{0}'' must match pattern ''{1}''."/>
            <message key="method.name.equals.class.name" value="方法名 ''{0}'' 不能与内部类名称相同."/>
        </module>

        <!-- 检查方法的参数名是否符合格式 -->
        <module name="MethodTypeParameterName">
            <property name="format" value="(^[A-Z][0-9]?)$|([A-Z][a-zA-Z0-9]*[T]$)"/>
            <message key="name.invalidPattern"
             value="方法参数名 ''{0}'' must match pattern ''{1}''."/>
        </module>

        <!-- 检查包名是否符合格式 -->
        <module name="PackageName">
            <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$"/>
            <message key="name.invalidPattern" value="包名 ''{0}'' 必须以 首字母小写,接下来的单词都以大写字母开头."/>
        </module>

        <!-- 检查参数名是否符合格式 -->
        <module name="ParameterName">
            <!-- <property name="format" value="^[a-z]([a-z0-9][a-zA-Z0-9]*)?$"/> -->
            <property name="format" value="(^[a-z][a-zA-Z0-9_]{0,19}$)"/>  
            <!-- <message key="name.invalidPattern" value="ParameterName ''{0}'' must match pattern ''{1}''."/> -->
            <message key="name.invalidPattern" value="参数名 ''{0}'' 必须以 首字母小写,接下来的单词都以大写字母开头,最大长度19."/>
        </module>

        <!-- 检查静态变量名是否符合格式 -->
        <module name="StaticVariableName">
            <property name="format" value="(^[A-Z0-9_]{0,19}$)"/>
            <message key="name.invalidPattern" value="静态变量名 ''{0}'' 必须 全部大写, 单词之间使用 _ 分隔,最大长度19."/>
            <!-- <message key="name.invalidPattern" value="Type name ''{0}'' must match pattern ''{1}''."/>  -->
        </module>

        <!-- 检查类名和接口名是否符合格式 -->
        <module name="TypeName">
            <property name="format" value="(^[A-Z][a-zA-Z0-9]{0,19}$)"/> 
            <message key="name.invalidPattern" value="类名接口名 ''{0}'' 必须以 首字母大写, 接下来的单词都以大写字母开头,最大长度19."/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Naming Conventions end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->


        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Regexp(正则表达式匹配不良编码习惯) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_regexp.html -->
        <!--
            RegexpMultiline         根据正则表达式检查多行
            RegexpOnFilename        根据正则表达式检查文件名
            RegexpSingleline        根据正则表达式检查单行是否有不良操作
            RegexpSinglelineJava    根据正则表达式查找java单行匹配的变体
        -->

        <!-- <module name="RegexpOnFilename"/> -->

        <!-- 检查  printStackTrace -->
        <module name="RegexpSinglelineJava">
            <property name="format" value="printStackTrace"/>
            <property name="message" value="bad practice of use printStackTrace"/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Regexp end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->


        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Size Violations(长度检查) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_sizes.html -->
        <!--
            AnonInnerLength             检查匿名内部类的长度。默认为20。
            ExecutableStatementCount    限制可执行代码片段的长度。默认为30。
            FileLength                  检查java文件的长度。默认为2000。
            LineLength                  检查代码行的长度。默认为80。
            MethodCount                 检查类里方法数量
            MethodLength                检查方法和构造函数的长度。默认为150。
            OuterTypeNumber             检查文件中外部级别的声明的类型数
            ParameterNumber             检查方法和构造函数的参数个数。默认为7。
        -->

        <!-- 匿名类的最大行数,缺省为20 -->
        <module name="AnonInnerLength">
            <property name="max" value="60"/>
        </module>

        <!-- 限制可执行代码片段的长度。默认为30。 -->
        <module name="ExecutableStatementCount">
            <property name="max" value="30"/>
        </module>

        <!-- 检查一行代码的长度 -->
        <module name="LineLength">
            <!-- 设置一行代码的最大长度 -->
            <property name="max" value="120"/>
            <!-- 定义可以忽略的格式 -->
            <property name="ignorePattern" value="^package.*|^import.*|a href|href|http://|https://|ftp://"/>
        </module>

        <!-- <module name="MethodCount" /> -->
        <!-- 方法不超过150行 -->
        <module name="MethodLength">
            <property name="tokens" value="METHOD_DEF" />
            <property name="max" value="150" />
            <message key="maxLen.method" value="方法长度 {0,number,integer} 行 (最大允许行数为 {1,number,integer})."/>
        </module>

        <!-- 检查文件中外部级别的声明的类型数 -->
        <module name="OuterTypeNumber">
            <property name="max" value="2"/>
        </module>

        <!-- 方法的参数个数不超过5个。 并且不对构造方法进行检查-->
        <module name="ParameterNumber">
            <property name="max" value="5" />
            <property name="tokens" value="METHOD_DEF" />
            <message key="maxParam" value="超过 {0,number,integer} 参数."/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Size Violations end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Checks for Whitespace(空白符检查) start @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <!-- See http://checkstyle.sourceforge.net/config_whitespace.html -->
        <!--
            EmptyForInitializerPad  检查空的初始化位置的空白。比如for循环中的初始化。
            EmptyForIteratorPad     检查for iterator语句是否使用空格
            EmptyLineSeparator      检查空白行分隔符
            FileTabCharacter        检查是否有Tab字符
            GenericWhitespace       检查<>和周围的空格
            MethodParamPad          检查方法签名之前的空白。
            NoLineWrap              检查所选语句是否没被换行，例如import包的语句
            NoWhitespaceAfter       检查分隔符后的空白。
            NoWhitespaceBefore      检查分隔符前的空白。
            OperatorWrap            检查操作符的空白规则。
            ParenPad                检查圆括号的空白规则。
            SeparatorWrap           检查带分隔线的换行
            SingleSpaceSeparator    检查非空格字符有不超过一个空格分隔
            TypecastParenPad        在类型转换时，不允许左圆括号右边有空格，也不允许与右圆括号左边有空格
            WhitespaceAfter         检查类型后是否包含空格
            WhitespaceAround        检查分隔符周围是否有空白。
        -->

        <!-- 检查是否有未初始化的循环变量 -->
        <module name="EmptyForInitializerPad"/>

        <!-- 检查for iterator语句是否使用空格 -->
        <module name="EmptyForIteratorPad">
            <property name="option" value="space"/>
        </module>

        <!-- 检查空白行分隔符 -->
        <module name="EmptyLineSeparator">
            <property name="allowNoEmptyLineBetweenFields" value="true"/>
        </module>

        <!-- 检查<>和周围的空格 -->
        <module name="GenericWhitespace">
            <message key="ws.followed"
             value="GenericWhitespace ''{0}'' is followed by whitespace."/>
             <message key="ws.preceded"
             value="GenericWhitespace ''{0}'' is preceded with whitespace."/>
             <message key="ws.illegalFollow"
             value="GenericWhitespace ''{0}'' should followed by whitespace."/>
             <message key="ws.notPreceded"
             value="GenericWhitespace ''{0}'' is not preceded with whitespace."/>
        </module>

        <!-- 允许方法名后紧跟左边圆括号"(" -->
        <module name="MethodParamPad" />

        <!-- 检查所选语句是否没被换行，例如import包的语句 -->
        <module name="NoLineWrap"/>

        <module name="NoWhitespaceAfter">
            <property name="allowLineBreaks" value="true"/>
        </module>

        <module name="NoWhitespaceBefore">
            <property name="allowLineBreaks" value="true"/>
        </module>

        <!-- 检查运算符是否在应在同一行 -->
        <module name="OperatorWrap">
        <!-- 下一行 -->
            <property name="option" value="NL"/>
            <property name="tokens" value="BAND, BOR, BSR, BXOR, DIV, EQUAL, GE, GT, LAND, LE, LITERAL_INSTANCEOF, LOR, LT, MINUS, MOD, NOT_EQUAL, PLUS, QUESTION, SL, SR, STAR, METHOD_REF "/>
        </module>

        <module name="ParenPad"/>

        <module name="SeparatorWrap">
            <property name="id" value="SeparatorWrapComma"/>
            <property name="tokens" value="COMMA"/>
            <property name="option" value="EOL"/>
        </module>

        <!-- 检查带分隔线的换行 -->
        <module name="SeparatorWrap">
            <!-- ELLIPSIS is EOL until https://github.com/google/styleguide/issues/258 -->
            <property name="id" value="SeparatorWrapEllipsis"/>
            <property name="tokens" value="ELLIPSIS"/>
            <property name="option" value="EOL"/>
        </module>
        <module name="SeparatorWrap">
            <!-- ARRAY_DECLARATOR is EOL until https://github.com/google/styleguide/issues/259 -->
            <property name="id" value="SeparatorWrapArrayDeclarator"/>
            <property name="tokens" value="ARRAY_DECLARATOR"/>
            <property name="option" value="EOL"/>
        </module>
        <module name="SeparatorWrap">
            <property name="id" value="SeparatorWrapMethodRef"/>
            <property name="tokens" value="METHOD_REF"/>
            <property name="option" value="nl"/>
        </module>

        <!-- <module name="SingleSpaceSeparator" />   -->

        <!-- 在类型转换时，不允许左圆括号右边有空格，也不允许与右圆括号左边有空格 -->
        <module name="TypecastParenPad" />

        <!-- <module name="WhitespaceAfter" />   -->

        <!-- 检查分隔符周围是否有空白。 -->
        <module name="WhitespaceAround">
            <property name="allowEmptyConstructors" value="true"/>
            <property name="allowEmptyMethods" value="true"/>
            <property name="allowEmptyTypes" value="true"/>
            <property name="allowEmptyLoops" value="true"/>
            <message key="ws.notFollowed"
             value="WhitespaceAround: ''{0}'' is not followed by whitespace. Empty blocks may only be represented as '{}' when not part of a multi-block statement (4.1.3)"/>
             <message key="ws.notPreceded"
             value="WhitespaceAround: ''{0}'' is not preceded with whitespace."/>
        </module>

        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@Checks for Whitespace end @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
    </module>
</module>
```



