��Spring 2.0�У�Pointcut�Ķ�������������֣�Pointcut��ʾʽ(expression)��Pointcutǩ��(signature)���������ȿ���execution��ʾʽ�ĸ�ʽ��
�����и���pattern�ֱ��ʾ���η�ƥ�䣨modifier-pattern?��������ֵƥ�䣨ret-type-pattern������·��ƥ�䣨declaring-type-pattern?����������ƥ�䣨name-pattern��������ƥ�䣨(param-pattern)�����쳣����ƥ�䣨throws-pattern?�������к�����š�?�����ǿ�ѡ�
�ڸ���pattern�п���ʹ�á�*������ʾƥ�����С���(param-pattern)�У�����ָ������Ĳ������ͣ�����������á�,������������Ҳ�����á�*������ʾƥ���������͵Ĳ�������(String)��ʾƥ��һ��String�����ķ�����(*,String)��ʾƥ�������������ķ�������һ�������������������ͣ����ڶ���������String���ͣ�������(..)��ʾ����������������
�����������������ӣ�
1��execution(* *(..))
��ʾƥ�����з���
2��execution(public * com. savage.service.UserService.*(..))
��ʾƥ��com.savage.server.UserService�����еĹ��з���
3��execution(* com.savage.server..*.*(..))
��ʾƥ��com.savage.server�������Ӱ��µ����з���
����execution��ʾʽ�⣬����within��this��target��args��Pointcut��ʾʽ��һ��Pointcut������Pointcut��ʾʽ��Pointcutǩ����ɣ����磺

execution(modifier-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)

//Pointcut��ʾʽ
@Pointcut("execution(* com.savage.aop.MessageSender.*(..))")
//Pointǩ��
private void log(){}
@Before("log()")


Pointcut����ʱ��������ʹ��&&��||��! ���㣬�磺
@Pointcut("execution(* com.savage.aop.MessageSender.*(..))")
private void logSender(){}

@Pointcut("execution(* com.savage.aop.MessageReceiver.*(..))")
private void logReceiver(){}

@Pointcut("logSender() || logReceiver()")
private void logMessage(){}
��������У�logMessage()��ƥ���κ�MessageSender��MessageReceiver�е��κη�����


�����Խ�һЩ���õ�Pointcut�ŵ�һ�����У��Թ�����Ӧ�ó���ʹ�ã��磺
package com.savage.aop;

import org.aspectj.lang.annotation.*;

public class Pointcuts {
@Pointcut("execution(* *Message(..))")
public void logMessage(){}

@Pointcut("execution(* *Attachment(..))")
public void logAttachment(){}

@Pointcut("execution(* *Service.*(..))")
public void auth(){}
}


��ʹ����ЩPointcutʱ��ָ����������������Pointcutǩ���Ϳ����ˣ��磺
package com.savage.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;

@Aspect
public class LogBeforeAdvice {
@Before("com.sagage.aop.Pointcuts.logMessage()")
public void before(JoinPoint joinPoint) {
System.out.println("Logging before " + joinPoint.getSignature().getName());
}
}


������XML Sechmaʵ��Adviceʱ�����Pointcut��Ҫ�����ã�����ʹ��<aop:pointcut></aop:pointcut>������Pointcut��Ȼ������Ҫʹ�����Pointcut�ĵط�����pointcut-ref���þ����ˣ��磺
<aop:config>
<aop:pointcut id="log"
expression="execution(* com.savage.simplespring.bean.MessageSender.*(..))"/>
<aop:aspect id="logging" ref="logBeforeAdvice">
<aop:before pointcut-ref="log" method="before"/>
<aop:after-returning pointcut-ref="log" method="afterReturning"/>
</aop:aspect>
</aop:config>


