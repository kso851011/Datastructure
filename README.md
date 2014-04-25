Datastructure
=============

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 연결 리스트를 구성하는 구조체를 선언

typedef struct t_node {
	char word[100];			// 단어
	int count;				// 단어 출현 빈도
	struct t_node* next;	// 다음 노드를 가리키는 포인터
} node;

// 파일 이름을 제외한 파일 경로를 저장하는 전역 변수

char file_route[256] = "articles";

// 단어가 있는지 찾아서 그 노드의 포인터를 반환하는 함수
// 없을 경우 NULL 포인터를 반환한다.
// search_word(찾는 단어, 연결 리스트의 시작 포인터)

node* search_word(char word[100] /*목표 단어*/, node* now /*찾을 노드(뒤쪽으로 쭉 반복적으로 검색*/) {
	if (now == NULL) return NULL;
	while (strcmp(word, now->word) != 0) {
		now = now->next;
		if (now == NULL) break;
	}
	return now;
}

// 파일에서 단어 하나를 입력받는 함수
// 정상적으로 입력받았을 경우 word_returned에 저장되며, 0을 반환한다.
// </TEXT>가 나와 파싱 범위를 벗어났을 경우 -1, 파일의 끝일 경우 0을 반환한다.

// 정상적인 입력 파일일 경우 if문에 걸릴 수가 없다.
// (이유는 </TEXT>가 나오면 파일의 끝이 나오기 전에 general_read_from_many_files 에서 파일이 닫힘

int read_word_in_a_file(char word_returned[100]/*읽은 단어 값을 넣을 변수*/, FILE* fp/*읽어올 파일의 포인터*/) {
	if (fscanf(fp, "%s", word_returned) == -1) {		// 파일을 읽음, EOF(파일 끝까지 다 읽음) 이 반환되면 끝냄
		return 0;
	}
	return 1;
}

// <TEXT> 를 찾아 파싱 시작 범위로 파일 커서를 옮기는 함수
// 찾았다면 1, 찾지 못했다면 0을 반환한다. (이 때, general_read_from_many_files에서 파일을 닫는다.)

int toTextTag(FILE* file/*탐색할 파일 포인터*/) {
	char buf[100] = { 0, };		// 파일에서 읽어온 단어를 저장할 변수
	int res = 0;				// 파일을 읽어왔을때 결과값을 저장하는 변수 
	while (strcmp(buf, "<TEXT>") != 0 /*<TEXT>가 읽히지 않았거나*/ && res != -1 /*res가 -1이 아닐 경우(즉, 파일의 끝까지 다 읽지 않은 경우)*/) {
		res = fscanf(file, "%s", buf);		// 파일에서 단어 하나를 읽어옴
	}

	if (res == -1) {				//파일의 끝까지 다 읽은 경우
		return 0;
	}
	return 1;
}

// 한개의 단어를 입력받는 함수.
// 정상적으로 받았다면 1을 반환하고, 단어는 word_segmented에 저장되어 createList에서 사용이 가능하다.
// 모든 파일에서 입력을 받았다면 0을 반환하여 createList의 루프를 종료한다.

int general_read_from_many_files(char word_segmented[100]/*찾아낸 단어를 넣을 변수*/) {
	static int filenumber = 880212;
	static FILE *fp;
	char filename[100];
	int res;

	// 이미 모든 파일을 받았을 경우에는 0을 반환

	if (filenumber > 880225) { return 0; }

	// 파일을 새로 열어야 하는 경우

	if (fp == NULL) {
		sprintf(filename, "%sap%d", file_route, filenumber);		// filename의 변수 안에 articles + ap + 파일넘버의 값을 출력한다 
		
		fp = fopen(filename, "rt");

		// 파일이 존재하지 않는다면, 다음 파일을 받는 것을 시도한다.

		if (fp == NULL) {
			
			filenumber++;
			return general_read_from_many_files(word_segmented);	// 다음 파일을 읽은 경우 재귀적으로 불러서 단어를 다시 읽기를 시도한다
		
		}

		// 파일을 연 후에, <TEXT> 부분을 찾아서 파일 포인터의 커서를 이동
		// 파일에 <TEXT> 태그가 없는 경우는 생각하지 않음 (항상 있다고 가정)

		toTextTag(fp);
	}

	// 정상적으로 단어 하나를 얻을 때까지 루프

	while (read_word_in_a_file(word_segmented, fp)) {	// read_word_in_a_file 함수에서 0이 반환될때까지 루프
		
		// 태그가 닫힌다면, 새로 태그를 열어서 커서를 위치시키고 다시 루프

		if (strcmp(word_segmented, "</TEXT>") == 0) {	// EOF가 반환되지 않았고 읽어온 값이 </TEXT>일 경우
			res = toTextTag(fp);

			// 새로운 <TEXT> 태그가 없다면 파일을 닫아야 하므로 루프 종료

			if (res == 0) {
				break;
			}
		}
		else {
			// 정상적으로 단어 하나를 얻었으므로 1을 반환

			return 1;
		}
	}

	// 파일 하나가 입력이 끝났으므로 파일을 닫고 번호를 증가

	fclose(fp);
	fp = NULL;
	filenumber++;

	// 새로 함수를 호출한 후 결과값을 그대로 반환

	return general_read_from_many_files(word_segmented);
}

// 연결 리스트를 아스키 코드 정렬 순으로 넣는다.
// strcmp를 사용했기 때문에 사전순은 아니다.
// 리스트를 정렬하려면 이 함수를 쓴다.

void insert_word_sort(node** head /*삽입할 위치*/, char word[100] /*삽입할 단어*/) {
	node* now = *head;
	node* before = NULL;
	node* add;
	int res;

	// 끝이 아닐때까지 돈다
	while (now->next != NULL) {
		// 현재 저장되어 있는 단어와, 넣고싶은 단어를 비교한다.
		res = strcmp(word, now->word);

		// 이미 이 단어가 연결 리스트에 있다면 카운트를 증가시킨다.

		if (res == 0) {
			now->count++;
			return;
		}

		// 현재 단어가 넣기를 원하는 단어보다 사전순으로 뒤라면 이 부분에 단어를 넣어준다.
		// before -> now 였던 연결 리스트를
		// before -> add -> now 로 바꾸고 add에 데이터를 넣고 종료한다.

		// 만약에 before가 없다는 뜻은 사전순으로 제일 앞이라는 뜻이므로
		// head가 가리키는 곳을 직접 변경하기 위해 더블 포인터로 인자를 받았으며
		// 이 경우는 head(now) -> ....를
		// add -> head(now) -> ... 구조로 바꾸며 add를 새로운 헤드로 바꿔준다. 
		// (*head) = add;가 해당 코드이다.
		// add->next의 경우는 add가 새로운 헤드, now가 이전 헤드이므로 저 코드를 사용해도 괜찮다.

		else if (res < 0) {
			add = (node*)malloc(sizeof(node));
			strcpy(add->word, word);
			add->count = 1;

			if (before != NULL) {
				before->next = add;
			}
			else {
				(*head) = add;
			}

			add->next = now;
			return;
		}
		else {
			before = now;
			now = now->next;
		}
	}

	// 끝까지 돌았는데도 자신보다 오름차순으로 뒤가 없다면, 
	// 이 단어는 현재까지 순서가 제일 뒤이므로 맨 뒤에 추가해준다.
	// 현재 마지막 포인터 (데이터가 없고 NULL을 가리키는 포인터) 에 단어를 넣고 카운트를 1로 해준다.

	strcpy(now->word, word);
	now->count = 1;

	// 새로 node를 만든다. 이 노드는 데이터가 없고 NULL을 가리키는 마지막을 의미하는 노드이다.
	// now->next->next는 새로 만든 노드가 NULL을 가리키게 한다는 뜻이다.

	now->next = (node*)malloc(sizeof(node));
	now->next->next = NULL;
}

// 연결 리스트를 생성하는 함수

node* createList() {
	node* head = (node*)malloc(sizeof(node));

	memset(head, 0, sizeof(head));
	head->next = NULL;

	int len = 0, sav = 0;
	char word[100];

	// general_read_from_many_files 를 호출하여 word 에 하나의 단어를 받아온다.
	// 반환 값이 1이라면 정상적으로 입력을 받아 word에 저장했다는 뜻이다.
	// 0일 경우에는 입력이 종료되었음을 알리며, 루프가 종료된다.

	while (general_read_from_many_files(word)) {

		// 한개의 단어를 받을 때마다 증가
		// 총 단어가 1470372 개이므로 147037 개마다 10%가 증가됨을 알림

		len++;
		if ((len / 14703) != sav) {		
			printf("\r");			// 줄 처음으로 이동하고 삭제
			printf("%d%%    ", ++sav);
			printf("\r");
		}
		
		// 단어를 추가한다. 중복 처리도 insert_word_loop 에서 처리한다.
		insert_word_sort(&head, word);

	}
	printf("\n");
	printf("\nlen = %d\n", len);
	return head;
}

	
// 연결 리스트를 메모리에서 해제하는 함수

void deleteList(node* head /*삭제할 리스트의 헤드*/) {
	node* temp;
	while (head != NULL) {		// 헤드가 NULL일때까지 (맨 뒤까지 갈때까지)
		temp = head;			// 임시로 head를 저장 
		head = head->next;		// head를 하나 뒤로 보낸후
		free(temp);				// 임시로 저장해 놓은 포인터를 이용해 메모리를 해제시켜준다
	}
	return;
}

int main(int argc /*입력받은 인자값의 개수*/ , char* argv[] /*입력받은 인자들*/) {
	node *head, *tar;
	char word[100];
	int i;

	// 프로그램을 실행할 때 인자로 경로가 넘어왔는 지 확인

	if (argc == 2) {
		strcpy(file_route, argv[1]);
	}
	/*else {

		// 같이 넘기지 않았다면 콘솔로 입력을 유도

		puts("파일경로를 입력해 주세요\n예> articles");
		scanf("%s", file_route);
	}*/

	// 마지막 문자에 역슬래시가 없을 경우 추가를 하여 차후에 파일 이름과 합칠 때 편하게 함

	if (file_route[strlen(file_route) - 1] != '\\') {
		strcat(file_route, "\\");
	}
	puts("prepare for searching...");

	// 연결 리스트를 생성

	head = createList();

	while (1) {
		puts("please enter the word to search(exit by enter $$$)");
		scanf("%s", word);
		if (strcmp(word, "$$$") == 0) break;

		tar = search_word(word, head);

		if (tar == NULL) {
			printf("there is no word %s in texts\n", word);
		}
		else {
			printf("count of word %s is %d\n", tar->word, tar->count);
		}
	}
	deleteList(head);
	puts("bye");
	return 0;
}
