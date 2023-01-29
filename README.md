# XmlPath-CheatSheet


Q: XMLPath Validation

Sample XML
<bookstore>
	<book category="cooking" test="passed">
		<title lang="en">The Nightingale</title>
		<author>Hannah</author>
		<year>2015</year>
		<price>
			<paperback>200</paperback>
			<hardcover>570</hardcover>
		</price>
	</book>
	<book category="children">
		<title lang="en">Harry Potter</title>
		<author>J K. Rowling</author>
		<year>2005</year>
		<price>29.99</price>
	</book>
</bookstore>

We are using the XML endpoint at : https://chercher.tech/sample/api/books.xml

We have to use a bath to get specific or arrays of values from the XML. I have listed a few below which may help you to understand things a little better.

root.parent.child.grandchild : get all grandchild element from the XML.
root.parant.child[0].grandchild : get a grand child which is present under first child (child[0]), here we use 0 as the array start from 0.
root.parent[0] : Extract the specific value we can also extract XML which is present under a tag, If you don't use the index then RestAssured will fresh all the matching elements and extract the example from them
it.@type == 'spcific value' : We can extract the attributes of other child elements when one of the child element has a specific value.
** : It is not possible all the time to write a complete XML a path to a specific item in such cases we can use ** to match any parent.

1. Extract the first match
Lets try to get a specific element from the XML response. We have to use the extract() method to get the value from the response, the response could be XML or it could be JSON.

path() method gets a value from the response body using the JsonPath or XmlPath syntax. REST Assured will automatically determine whether to use JsonPath or XmlPath based on the content-type of the response.

import org.junit.jupiter.api.Test;
import io.restassured.RestAssured;
public class XMLValuesExtraction {
	@Test
	void sampleTest() {
		String book = RestAssured.given().when()
		.get("https://chercher.tech/sample/api/books.xml")
		.then().extract().path("bookstore.book.title");
		System.out.println(book);
	}
}

The Output:
The Nightingale

2. Get Nodes based on a specific value:
we can get the values of the sibling/child elements based on a specific element. For example, consider, you want to retrieve the name of the author whose books are in a specific category, then we can use this kind of XML path.

@Test
void sampleTest() {
	String book = RestAssured.given().when()
	.get("https://chercher.tech/sample/api/books.xml")
	.then().extract().path("bookstore.book.findAll { it.@category == 'cooking' }.year");
	System.out.println(book);
}
The output:
2015


3. Deep Search:
You can use the deep search when you are not sure of the part elements but you know only the values, then in such cases, you can use the deep search. A deep search will find all the matching elements irrespective of their location in the XML.

This also helps in reducing the length of the XML path that we were writing.

@Test
void sampleTest() {
	String book = RestAssured.given().when()
	.get("https://chercher.tech/sample/api/books.xml")
	.then().extract().path("**.findAll { it.@category == 'cooking' }.year");
	System.out.println(book);
}
The output:
2015

When there is more than one matching element then you may see the below error.

java.lang.ClassCastException: io.restassured.internal.path.xml.NodeChildrenImpl cannot be cast to java.base/java.lang.String

In such cases, please use the Type as NodeChildrenImpl instead of String

@Test
void sampleTest() {
	NodeChildrenImpl book = RestAssured.given().when()
	.get("https://chercher.tech/sample/api/books.xml")
	.then().extract().path("bookstore.book.findAll { it.@category == 'cooking' }.price");
	System.out.println(book);
}

4. Extract all the matches from XML 
Similar to extracting a single value we can extract multiple values as well. To extract multiple values we have to use another class called NodeChildrenImpl.

@Test
void sampleTest() {
	NodeChildrenImpl books = RestAssured.given().when()
	.get("https://chercher.tech/sample/api/books.xml")
	.then().extract().path("bookstore.book.title");
	System.out.println(books);
}
The output ( returned as a single string):
The NightingaleHarry Potter

Important Methods from NodeChildrenImpl:
Using NodeChildrenImpl class, we can perform other operations on the examine extracted. I have listed a few below.

get(int index): Get the value of the specific index
size(): Which is the size of the retrieved elements
isEmpty(): Checks whether anything got retrieved or zero elements present; if 0 elements present it will return true otherwise false
list(): By default, NodeChildrenImpl returns a string format but when we use list we can get the output in list format

import org.junit.jupiter.api.Test;
import io.restassured.RestAssured;
import io.restassured.internal.path.xml.NodeChildrenImpl;
public class XMLValuesExtraction {
	@Test
	void sampleTest() {
		NodeChildrenImpl books = RestAssured.given().when()
		.get("https://chercher.tech/sample/api/books.xml")
		.then().extract().path("bookstore.book.title");

		System.out.println("just single string: "+ books);
		System.out.println("spcific index : "+ books.get(0));
		System.out.println("is empty : "+ books.isEmpty());
		System.out.println("size : "+ books.size());
		System.out.println("list : "+ books.list());
	}
}



5. NodeChildrenImpl on a subset of XML:
In the above program, we have seen how to use NodeChildrenImpl for simple operations; now let's apply this class on a subset of XML.

When we try to fetch a subset of XML, we might need to extract the values from the subset of XML. In such cases, we can use NodeChildrenImpl

Example to fetch the subset of XML:

@Test
void sampleTest() {
	NodeChildrenImpl books = RestAssured.given().when()
	.get("https://chercher.tech/sample/api/books.xml")
	.then().extract().path("bookstore.book");

	System.out.println("list : "+ books);
	System.out.println("list : "+ books.get(0));
}


Don't be scared that in the above output we have not received the subset XML; we have received subset XML but the NodeChildrenImpl class only has given the values present in the subset XML because NodeChildrenImpl class handle tags internally.

So we will be performing operations on the first match (Please read the program as well for better understanding).

name(): Gets the tag name of the element(subset)
attributes(): fetches all attributes present in the subset XML of the parent element
getAttribute(): Fetches the value for the given attribute from the root element of the subset XML
get(): Gets the value present in the given tag name.
children(): All the tags present in the root tag of the subset
children().get(index): Gets the element based on the index inside the subset of XML
getNode(): Fetches another subset of XML, on which we can perform the above mentioned all operations



@Test
void sampleTest() {
	NodeChildrenImpl books = RestAssured.given().when()
	.get("https://chercher.tech/sample/api/books.xml")
	.then().extract().path("bookstore.book");

	System.out.println("name : "+ books.get(0).name());
	System.out.println("attributes : "+ books.get(0).attributes());
	System.out.println("getAttribute : "+ books.get(0).getAttribute("category"));
	System.out.println("get : "+ books.get(0).get("year"));
	System.out.println("children : "+ books.get(0).children());
	System.out.println("children : "+ books.get(0).children().get("price"));
	System.out.println("getNode : "+ books.get(0).getNode("price").children().get(0));
}


Q: What is a Resource?
REST architecture treats every content as a resource. These resources can be Text Files, Html Pages, Images, Videos or Dynamic Business Data. REST Server simply provides access to resources and REST client accesses and modifies the resources. Here each resource is identified by URIs/ Global IDs. REST uses various representations to represent a resource where Text, JSON, XML. The most popular representations of resources are XML and JSON.
