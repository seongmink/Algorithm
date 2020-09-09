# Trie

문자열을 보다 빨리 조회하는 방법 중 하나이며, 텍스트 자동 완성 기능과 같이 문자열을 저장하고 탐색하는데 유용하다.

"TRIE"라는 문자열을 검색할 때, 루트 노드부터 [T] -> [R] -> [I] -> [E] 순으로 탐색한다. (전화번호부 검색과 같다) 루트 노드는 특정 문자를 의미하지 않고, 자식 노드(문자)만 가지고 있으며, 그 자식 노드에서 이어지는 다른 문자의 정보들도 가지고 있다. 즉, **루트 노드를 제외한 노드의 자손들은 해당 노드와 공통 접두어를 포함한다.**

정리하면 Trie는 정렬된 트리 구조이며, 자식 노드를 Map<Key, Value> 형태로 가지고 있다. 문자열 탐색시 O(N)의 속도를 가져 매우 빠르게 탐색이 가능하나, 빠른 속도만큼 공간복잡도가 높기 때문에 주의해서 사용해야 한다.

## 코드

##### TrieNode

```java
import java.util.HashMap;
import java.util.Map;

public class TrieNode {

	private Map<Character, TrieNode> childNodes = new HashMap<>();
	private boolean isLastChar;

	public Map<Character, TrieNode> getChildNodes() {
		return this.childNodes;
	}

	public boolean isLastChar() {
		return this.isLastChar;
	}
    
	public void setIsLastChar(boolean isLastChar) {
		this.isLastChar = isLastChar;
	}
}
```

먼저 TrieNode 클래스를 구현한다.

childNodes는 하위 노드들을 담을 Map이고, isLastChar은 마지막 문자 여부이다. 예를들어 "TRIE"라는 문자열에서 E가 마지막 문자이기 때문에, E의 isLastChar를 true로 변경하여 문자열의 마지막을 알 수 있다.

그리고 childNodes의 getter와 isLastChar의 getter/setter를 생성한다. 마지막 글자 여부는 추후 노드 삭제하는 과정에서 변경이 필요하기 때문에 setter를 추가적으로 생성한다.

##### Trie

```java
public class Trie {

	private TrieNode rootNode;
	
	public Trie() {
		rootNode = new TrieNode();
	}
	
	public void add(String word) {
		TrieNode nowNode = this.rootNode;
		
		for (int i = 0; i < word.length(); i++) {
			nowNode = nowNode.getChildNodes().computeIfAbsent(word.charAt(i), s -> new TrieNode());
		}
		nowNode.setLastChar(true);
	}
	
	public boolean contains(String word) {
		TrieNode nowNode = this.rootNode;
		
		for (int i = 0; i < word.length(); i++) {
			TrieNode node = nowNode.getChildNodes().get(word.charAt(i));
			if (node == null)
				return false;
			
			nowNode = node;
		}
		
		return nowNode.isLastChar();
	}
	
	public void delete(String word) {
		delete(rootNode, word, 0);
	}

	public void delete(TrieNode nowNode, String word, int index) {
		char character = word.charAt(index);
		
		if (!nowNode.getChildNodes().containsKey(character))
			return;
		
		TrieNode childNode = nowNode.getChildNodes().get(character);
		index++;
		
		if (index == word.length()) {
			if (!childNode.isLastChar())
				return;
			
			childNode.setLastChar(false);
			if (childNode.getChildNodes().isEmpty())
				nowNode.getChildNodes().remove(character);
		} else {
			delete(childNode, word, index);

			if (!childNode.isLastChar() && childNode.getChildNodes().isEmpty())
				nowNode.getChildNodes().remove(character);
		}
	}
}
```

- #### add(String word) 메소드

  입력받은 단어의 각 알파벳을 계층구조의 자식노드로 만들어 넣는다. 이 때, 이미 같은 알파벳이 존재하면 공통 접두어 부분까지는 생성하지 않는다. 즉, 해당 계층 문자의 자식노드가 존재하지 않을 때에만 자식 노드를 생성한다.

- #### contains(String word) 메소드

  특정 단어가 Trie에 존재하는지를 확인하기 위해서는 다음 두 가지 조건을 만족시켜야 한다.

  - 루트노드부터 순서대로 알파벳이 일치하는 자식노드들이 존재 할 것

  - 해당 단어의 마지막 글자에 해당하는 노드의 isLastChar가 true일 것(마지막 문자가 true가 아니면 해당 문자열이 아닌 다른 문자열로 생성된 트리 구조이다)

- #### delete(String word) 메소드

  주의할 점은 노드들이 부모노드의 정보를 가지고 있지 않기 때문에, 하위 노드로 내려가며 삭제 대상 단어를 탐색하고 다시 올라오며 삭제하는 과정이 재귀로 구현된다는 점이다.

  탐색 방향은 **부모 노드 -> 자식노드**이지만 삭제 방향은 **자식 노드 -> 부모 노드**이다.

  삭제 진행은 마지막 글자에서 부모 노드 방향으로 되돌아 오는 과정에서 진행된다는 점에 유의하여 다음 삭제 조건을 만족해야 한다.

  - 자식 노드를 가지고 있지 않아야 한다.
  - 삭제를 시작하는 첫 노드는 isLastChar == true이어야 한다.
  - 삭제를 진행하던 중에는 isLastChar == false이어야 한다.