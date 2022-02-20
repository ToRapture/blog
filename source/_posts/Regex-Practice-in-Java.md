---
title: Regex Practice in Java
date: 2022-02-20 13:17:15
tags:
- Java
- Regex
---

```java
import static org.junit.Assert.assertArrayEquals;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;

import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.junit.Test;

public class AppTest {

    private static List<String> getGroups(Matcher m, int groups) {
        List<String> result = new ArrayList<String>();
        while (m.find()) {
            for (int i = 0; i <= groups; i++) {
                result.add(m.group(i));
            }
        }
        return result;
    }

    @Test
    public void testMain() {
        // Returns only true if the WHOLE string can be matched.
        assertTrue("".matches(""));
        assertTrue("".matches("^$"));

        assertTrue("abc".matches("abc"));
        assertTrue("abc".matches("^abc$"));

        assertTrue("abc".matches("\\babc"));
        assertTrue("abc".matches("^\\babc$"));
        assertFalse("abc".matches("^a\\bbc$"));

        assertTrue("abc-112=112-abc".matches("^(\\w+)-(\\d+)=\\2-\\1$"));

        assertTrue("He2LlO".matches("(?i)he\\dllo"));
        assertFalse("He2LlO".matches("he\\dllo"));

        assertTrue("abcabcabc".matches("^(abc)+$"));
        assertEquals("2211-abc+=1145-bba", "abc2211+=bba1145".replaceAll("([a-z]+)(\\d+)", "$2-$1"));
        assertEquals("2211-abc+=bba1145", "abc2211+=bba1145".replaceFirst("([a-z]+)(\\d+)", "$2-$1"));
        assertEquals("abc2211-=bba1145", "abc2211+=bba1145".replace("+=", "-="));

        assertArrayEquals(new String[] { "abcab", "wjio", "ksldm", "xf", "w", "000wefj" },
                "abcabc1wjion2ksldmf3xfe9wf0000wefj".split("[a-z]\\d"));
        assertArrayEquals(new String[] { "abc\nab", "\nwjio", "  ksld  m", "x\nf", "w", "0\n00wefj" },
                "abc\nabc1\nwjion2  ksld  mf3x\nfe9wf00\n00wefj".split("[a-z]\\d"));

        Pattern p = Pattern.compile("(\\w+?)-(\\d+)");
        Matcher m = p.matcher("123-123444-444gg-212=====bbbbb111-bb1111-bb-22");
        assertArrayEquals(
                new String[] { "123-123444", "123", "123444", "444gg-212", "444gg", "212", "bb-22", "bb", "22" },
                getGroups(m, 2).toArray());

        Pattern q = Pattern.compile("(\\w+?)-(\\d+?)");
        Matcher n = q.matcher("123-123444-444gg-212=====bbbbb111-bb1111-bb-22");
        assertArrayEquals(
                new String[] { "123-1", "123", "1", "23444-4", "23444", "4", "44gg-2", "44gg", "2", "bb-2", "bb", "2" },
                getGroups(n, 2).toArray());

        Pattern r = Pattern.compile("\\d-\\d");
        assertArrayEquals(new String[] { "2-3", "5-2", "1-3" },
                getGroups(r.matcher("ab 2-3\n2343\r2 \t55-22\n\r3331-3\r\n-33"), 0).toArray());
    }
}

```