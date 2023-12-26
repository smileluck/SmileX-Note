[toc]

---

# 前言

`RequestCondition` 是 `Spring MVC` 对一个请求匹配条件的概念模型。最终的实现类可能是针对以下情况之一：路径匹配，头部匹配，请求参数匹配，可产生`MIME`匹配，可消费`MIME`匹配，请求方法匹配，或者是以上各种情况的匹配条件的一个组合。 

# 执行过程

大致执行过程：（个人理解）

1. 接收到请求后，通过`AbstractHandlerMethodMapping.lookupHandlerMethod`方法遍历接口
2. 通过 `RequestCondition.getMatchingCondition`方法获取出匹配的 `RequestCondition`并包装成 `Match`
3. 然后对 `List<Match>`匹配列表进行比较，通过 `RequestCondition.compareTo` 确定最后的最匹配的
4. 如果出现匹配程度一致的，抛出异常 `IllegalStateException`

## AbstractHandlerMethodMapping

```java
@Nullable
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
    List<Match> matches = new ArrayList<>();
    List<T> directPathMatches = this.mappingRegistry.getMappingsByUrl(lookupPath);
    if (directPathMatches != null) {
        addMatchingMappings(directPathMatches, matches, request);
    }
    if (matches.isEmpty()) {
        // No choice but to go through all mappings... 
        // 获取出匹配的路径条件
        addMatchingMappings(this.mappingRegistry.getMappings().keySet(), matches, request);
    }
    if (!matches.isEmpty()) {
			Match bestMatch = matches.get(0);
        	// 如果匹配数量大于1
			if (matches.size() > 1) {
				Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
                // 进行排序，执行 RequestCondition.compareTo
				matches.sort(comparator);
				bestMatch = matches.get(0);
				if (logger.isTraceEnabled()) {
					logger.trace(matches.size() + " matching mappings: " + matches);
				}
                // 如果是CORS预检查
				if (CorsUtils.isPreFlightRequest(request)) {
					return PREFLIGHT_AMBIGUOUS_MATCH;
				}
				Match secondBestMatch = matches.get(1);
                // 如果两者比较一致，抛出异常。
				if (comparator.compare(bestMatch, secondBestMatch) == 0) {
					Method m1 = bestMatch.handlerMethod.getMethod();
					Method m2 = secondBestMatch.handlerMethod.getMethod();
					String uri = request.getRequestURI();
					throw new IllegalStateException(
							"Ambiguous handler methods mapped for '" + uri + "': {" + m1 + ", " + m2 + "}");
				}
			}
        	// 设置最匹配的路径
			request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.handlerMethod);
			handleMatch(bestMatch.mapping, lookupPath, request);
			return bestMatch.handlerMethod;
    }

    else {
        // 没有匹配出来
        return handleNoMatch(this.mappingRegistry.getMappings().keySet(), lookupPath, request);
    }
}
```



# 源代码

## `RequestCondition`定义

```java
package org.springframework.web.servlet.mvc.condition;

import javax.servlet.http.HttpServletRequest;

import org.springframework.lang.Nullable;

public interface RequestCondition<T> {

	// 将此条件和另外一个请求匹配条件合并，合并逻辑由实现类决定。
	T combine(T other);

    // 检查条件是否与当前请求实例相匹配，如果不匹配返回null，
	// 如果匹配，生成一个新的请求匹配条件，该新的请求匹配条件是当前请求匹配条件
	// 举个例子来讲，如果当前请求匹配条件是一个路径匹配条件，包含多个路径匹配模板，
	// 并且其中有些模板和指定请求request匹配，那么返回的新建的请求匹配条件将仅仅
	// 包含和指定请求request匹配的那些路径模板。
    // 对于 Cors预检查请求，条件应与请求实例相匹配。如果不匹配，应确保返回一个示例空内容，不会导致匹配失败。
	@Nullable
	T getMatchingCondition(HttpServletRequest request);

	// 针对指定的请求对象request比较两个请求匹配条件。
	// 该方法假定被比较的两个请求匹配条件都是针对该请求对象request调用了
	// #getMatchingCondition方法得到的，这样才能确保对它们的比较
	// 是针对同一个请求对象request，这样的比较才有意义(最终用来确定谁是
	// 更匹配的条件)。
    // 
	int compareTo(T other, HttpServletRequest request);

}
```

 由接口源代码可以看出，接口`RequestCondition`是一个泛型接口。事实上，它的**泛型参数`T`通常也会是一个`RequestCondition`对象**。 

## `AbstractRequestCondition` 定义

框架对接口`RequestCondition`有一组具体实现类。对这些具体实现类的一些通用逻辑，比如`equals`,`hashCode`和`toString`，是被放到抽象基类`AbstractRequestCondition`来实现的。同时`AbstractRequestCondition`还通过`protected`抽象方法约定了实现类其他的一些内部通用逻辑，具体如下所示：

```java
package org.springframework.web.servlet.mvc.condition;

import java.util.Collection;
import java.util.Iterator;

import org.springframework.lang.Nullable;

/**
 * @param <T> the type of objects that this RequestCondition can be combined
 * with and compared to
 */
public abstract class AbstractRequestCondition<T extends AbstractRequestCondition<T>> implements RequestCondition<T> {

	/**
	 * 当前请求匹配条件对象是否内容为空
	 * @return  true if empty; false otherwise
	 */
	public boolean isEmpty() {
		return getContent().isEmpty();
	}

	/**
	 * 一个请求匹配条件可能由多个部分组成，这些组成部分被包装成一个名为 content 的集合
	 * 比如 ：
	 * 对于请求路径匹配条件，可能有多个 URL pattern,
	 * 对于请求方法匹配条件，可能有多个 HTTP request method,
	 * 对于请求参数匹配条件，可能有多个 param 表达式 .
	 * @return a collection of objects, never  null ， 可能为空集合
	 */
	protected abstract Collection<?> getContent();

	/**
	 * The notation to use when printing discrete items of content.
	 * 将该条件作为字符串展示时，各个组成部分之间的中缀标识符。比如 "||" 或者 "&&" 等。
	 * For example {@code " || " for URL patterns or {@code " && "} for param expressions.
	 */
	protected abstract String getToStringInfix();


	// equlas 实现 
	@Override
	public boolean equals(@Nullable Object other) {
		if (this == other) {
			return true;
		}
		if (other == null || getClass() != other.getClass()) {
			return false;
		}
		return getContent().equals(((AbstractRequestCondition<?>) other).getContent());
	}

	// hashCode 实现
	@Override
	public int hashCode() {
		return getContent().hashCode();
	}

	// toString 实现
	@Override
	public String toString() {
		StringBuilder builder = new StringBuilder("[");
		for (Iterator<?> iterator = getContent().iterator(); iterator.hasNext();) {
			Object expression = iterator.next();
			builder.append(expression.toString());
			if (iterator.hasNext()) {
				builder.append(getToStringInfix());
			}
		}
		builder.append("]");
		return builder.toString();
	}

}
```

继承 `AbstractRequestCondition` 的具体实现类

| 实现类                         | 简介               |
| ------------------------------ | ------------------ |
| PatternsRequestCondition       | 路径匹配条件       |
| RequestMethodsRequestCondition | 请求方法匹配条件   |
| ParamsRequestCondition         | 请求参数匹配条件   |
| HeadersRequestCondition        | 头部信息匹配条件   |
| ConsumesRequestCondition       | 可消费MIME匹配条件 |
| ProducesRequestCondition       | 可生成MIME匹配条件 |

框架还提供了一个实现类`RequestConditionHolder`，这是一个匹配条件持有器，用于持有某个`RequestCondition`对象。如果你想持有一个`RequestCondition`对象,但其类型事先不可知，那么这种情况下该工具很有用。但要注意的是如果要合并或者比较两个`RequestConditionHolder`对象，也就是二者所持有的`RequestCondition`对象，那么二者所持有的`RequestCondition`对象必须是类型相同的，否则会抛出异常`ClassCastException`。
