# Log4j BurstFilter

* log4j2 에 해당되는 내용이다

## BurstFilter

* http://logging.apache.org/log4j/2.x/manual/filters.html#BurstFilter
* https://github.com/apache/logging-log4j2/blob/master/log4j-core/src/main/java/org/apache/logging/log4j/core/filter/BurstFilter.java

* 많은 트래픽을 처리하는 애플리케이션에서 에러 로그가 너무 많이 발생하면 disk io 나 용량 관련 추가적인 문제가 발생할 수 있다.
* 로그 라이브러리로 log4j2 를 사용한다면 `BurstFilter` 를 적용해서 시간당 최대 로그 수를 제한할 수 있다.

### Parameters
* level 
    * burst filter 를 거칠 로그 레벨 제한
    * 만약 warn 으로 설정했다면 warn 보다 낮은 레벨의 로그들은 (info, debug ...) 모두 시간당 쓰기량에 제한을 받는다.
* rate
    * 초당 허용할 로그 수의 평균값
* maxBurst
    * 초당 허용할 로그 수의 최댓값


### rate & maxBurst

* maxBurst : 1000, rate : 100 이라고 할 때,
* 초당 100 개 미만의 로그가 생성될 때는 아무 문제가 없다.
* 애플리케이션 구동 후 처음으로 초당 1000 개의 로그가 생성되었을 때는 다음 9초 동안은 아무 로그도 생성되지 않는다. (초당 100 개가 평균이려면 1000개의 로그는 10초 동안 생성된 양이어야 하기 떄문)

## for Log4j 1.2

* log4j 1.2 용으로 포팅한 코드
* 달라진 부분은 생성자가 없어졌다는 것과 option 을 가져오는 부분, 그리고 decide 인터페이스로 맞춘것 정도.
* 아래 properties 파일 예제도 추가함

* org.apache.log4j.varia.BurstFilter
```java
package org.apache.log4j.varia;

/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache license, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the license for the specific language governing permissions and
 * limitations under the license.
 */

import org.apache.log4j.Level;
import org.apache.log4j.helpers.OptionConverter;
import org.apache.log4j.spi.Filter;
import org.apache.log4j.spi.LoggingEvent;

import java.util.Queue;
import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

/**
 * The <code>BurstFilter</code> is a logging filter that regulates logging traffic.
 *
 * <p>
 * Use this filter when you want to control the maximum burst of log statements that can be sent to an appender. The
 * filter is configured in the log4j configuration file. For example, the following configuration limits the number of
 * INFO level (as well as DEBUG and TRACE) log statements that can be sent to the console to a burst of 100 with an
 * average rate of 16 per second. WARN, ERROR and FATAL messages would continue to be delivered.
 * </p>
 * <code>
 * &lt;Console name="console"&gt;<br>
 * &nbsp;&lt;PatternLayout pattern="%-5p %d{dd-MMM-yyyy HH:mm:ss} %x %t %m%n"/&gt;<br>
 * &nbsp;&lt;filters&gt;<br>
 * &nbsp;&nbsp;&lt;Burst level="INFO" rate="16" maxBurst="100"/&gt;<br>
 * &nbsp;&lt;/filters&gt;<br>
 * &lt;/Console&gt;<br>
 * </code><br>
 */

public class BurstFilter extends Filter {

    private static final long NANOS_IN_SECONDS = 1000000000;

    private static final int DEFAULT_RATE = 10;

    private static final int DEFAULT_RATE_MULTIPLE = 100;

    private static final int HASH_SHIFT = 32;

    /**
     * Level of messages to be filtered. Anything at or below this level will be
     * filtered out if <code>maxBurst</code> has been exceeded. The default is
     * WARN meaning any messages that are higher than warn will be logged
     * regardless of the size of a burst.
     */
    private Level level = Level.WARN;

    private Long burstInterval;

    private Float rate = (float) DEFAULT_RATE;

    private Long maxBurst = (long) DEFAULT_RATE * DEFAULT_RATE_MULTIPLE;

    private final DelayQueue<LogDelay> history = new DelayQueue<>();

    private final Queue<LogDelay> available = new ConcurrentLinkedQueue<>();

    static LogDelay createLogDelay(final long expireTime) {
        return new LogDelay(expireTime);
    }

    public BurstFilter() {
        super();
        setBurstInterval();
    }

    public String getLevel()  {
        return level == null ? null : level.toString();
    }

    public void setLevel(String level) {
        this.level = OptionConverter.toLevel(level, null);
    }

    private void setBurstInterval() {
        this.burstInterval = (long) (NANOS_IN_SECONDS * (maxBurst / rate));
        available.clear();
        for (int i = 0; i < maxBurst; ++i) {
            available.add(createLogDelay(0));
        }
    }

    public String getRate() {
        return rate.toString();
    }

    public void setRate(String rate) {
        this.rate = (float) OptionConverter.toInt(rate, DEFAULT_RATE);
        setBurstInterval();
    }

    public String getMaxBurst() {
        return maxBurst.toString();
    }

    public void setMaxBurst(String maxBurst) {
        this.maxBurst = (long) OptionConverter.toInt(maxBurst, DEFAULT_RATE * DEFAULT_RATE_MULTIPLE);
        setBurstInterval();
    }

    /**
     * Decide if we're going to log <code>event</code> based on whether the
     * maximum burst of log statements has been exceeded.
     *
     * @return NEUTRAL if the filter passes, DENY otherwise.
     */
    public int decide(LoggingEvent event) {
        if (this.level.toInt() <= event.getLevel().toInt()) {
            LogDelay delay = history.poll();
            while (delay != null) {
                available.add(delay);
                delay = history.poll();
            }
            delay = available.poll();
            if (delay != null) {
                delay.setDelay(burstInterval);
                history.add(delay);
                return NEUTRAL;
            }
            return DENY;
        }
        return NEUTRAL;

    }

    /**
     * Returns the number of available slots. Used for unit testing.
     * @return The number of available slots.
     */
    public int getAvailable() {
        return available.size();
    }

    /**
     * Clear the history. Used for unit testing.
     */
    public void clear() {
        for (final LogDelay delay : history) {
            history.remove(delay);
            available.add(delay);
        }
    }

    @Override
    public String toString() {
        return "level=" + level.toString() + ", interval=" + burstInterval + ", max=" + history.size();
    }

    /**
     * Delay object to represent each log event that has occurred within the timespan.
     *
     * Consider this class private, package visibility for testing.
     */
    private static class LogDelay implements Delayed {

        LogDelay(final long expireTime) {
            this.expireTime = expireTime;
        }

        private long expireTime;

        public void setDelay(final long delay) {
            this.expireTime = delay + System.nanoTime();
        }

        @Override
        public long getDelay(final TimeUnit timeUnit) {
            return timeUnit.convert(expireTime - System.nanoTime(), TimeUnit.NANOSECONDS);
        }

        @Override
        public int compareTo(final Delayed delayed) {
            final long diff = this.expireTime - ((LogDelay) delayed).expireTime;
            return Long.signum(diff);
        }

        @Override
        public boolean equals(final Object o) {
            if (this == o) {
                return true;
            }
            if (o == null || getClass() != o.getClass()) {
                return false;
            }

            final LogDelay logDelay = (LogDelay) o;

            if (expireTime != logDelay.expireTime) {
                return false;
            }

            return true;
        }

        @Override
        public int hashCode() {
            return (int) (expireTime ^ (expireTime >>> HASH_SHIFT));
        }
    }
}
```

* log4j.properties
```
log4j.rootLogger=INFO, console

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p %-60c %x - %m%n
log4j.appender.console.filter.burst=org.apache.log4j.varia.BurstFilter
log4j.appender.console.filter.burst.level=WARN
log4j.appender.console.filter.burst.rate=10
log4j.appender.console.filter.burst.maxBurst=100
```