from copy import deepcopy
import numpy as np
import time
 
# 현재 상태들을 입력받아 목표 상태로 가는 최적의 경로를 계산합니다.
def bestsolution(state):
    bestsol = np.array([], int).reshape(-1, 9)
    count = len(state) - 1
    while count != -1:
        bestsol = np.insert(bestsol, 0, state[count]['puzzle'], 0)
        count = (state[count]['parent'])
    return bestsol.reshape(-1, 3, 3)
 
        
# 이 함수는 현재 반복 상태가 이전에 탐색되었는지 여부를 확인하여 고유한지 검사합니다.
def all(checkarray):
    set=[]
    for it in set:
         for checkarray in it:
             return 1
         else:
             return 0
 
 
# 퍼즐(시작 상태)의 각 숫자와 목표 상태 간의 맨해튼 거리 비용을 계산합니다.
def manhattan(puzzle, goal):
    a = abs(puzzle // 3 - goal // 3)
    b = abs(puzzle % 3 - goal % 3)
    mhcost = a + b
    return sum(mhcost[1:])
 
 
 
# 현재 상태에서 목표 상태와 비교하여 잘못 배치된 타일의 개수를 계산합니다.
def misplaced_tiles(puzzle,goal):
    mscost = np.sum(puzzle != goal) - 1
    return mscost if mscost > 0 else 0
        
 
# 3[참일 경우] if [표현식] else [거짓일 경우] 
 
 
# 목표 상태 또는 초기 상태의 각 값의 좌표를 식별합니다.
def coordinates(puzzle):
    pos = np.array(range(9))
    for p, q in enumerate(puzzle):
         pos[q] = p
    return pos
 
 
 
# 맨해튼 휴리스틱을 사용한 8 퍼즐 평가 시작
def evaluvate(puzzle, goal):
    steps = np.array([('up', [0, 1, 2], -3),('down', [6, 7, 8],  3),('left', [0, 3, 6], -1),('right', [2, 5, 8],  1)],
                 dtype =  [('move',  str, 1),('position', list),('head', int)])
 
    dtstate = [('puzzle',  list),('parent', int),('gn',  int),('hn',  int)]
     
    # 부모, gn, hn을 초기화합니다. 여기서 hn은 맨해튼 거리 함수 호출 결과입니다.
    costg = coordinates(goal)
    parent = -1
    gn = 0
    hn = manhattan(coordinates(puzzle), costg)
    state = np.array([(puzzle, parent, gn, hn)], dtstate)
 
    # 위치를 키로, f(n)을 값으로 하는 우선순위 큐를 사용합니다.
    dtpriority = [('position', int),('fn', int)]
    priority = np.array( [(0, hn)], dtpriority)
 
 
    while 1:
        priority = np.sort(priority, kind='mergesort', order=['fn', 'position'])     
        position, fn = priority[0]                                                 
        priority = np.delete(priority, 0, 0)
        # 병합 정렬을 사용하여 우선순위 큐를 정렬합니다. 탐색할 첫 번째 요소를 선택하고, 해당 요소를 큐에서 제거합니다.
        puzzle, parent, gn, hn = state[position]
        puzzle = np.array(puzzle)
        # 입력된 퍼즐에서 빈 칸(0)을 찾습니다.
        blank = int(np.where(puzzle == 0)[0][0])
        gn = gn + 1
        c = 1
        start_time = time.time()
        for s in steps:
            c = c + 1
            if blank not in s['position']:
                # 현재 상태를 복사하여 새로운 상태를 생성합니다.
                openstates = deepcopy(puzzle)
                openstates[blank], openstates[blank + s['head']] = openstates[blank + s['head']], openstates[blank]
                # 노드가 이전에 탐색되었는지 확인하기 위해 all 함수가 호출됩니다.
                if ~(np.all(list(state['puzzle']) == openstates, 1)).any():
                    end_time = time.time()
                    if (( end_time - start_time ) > 2):
                        print(" The 8 puzzle is unsolvable ! \n")
                        exit
                    # 맨해튼 함수를 호출하여 비용을 계산합니다.
                    hn = manhattan(coordinates(openstates), costg)
                    # 새로운 상태를 생성하여 리스트에 추가합니다.
                    q = np.array([(openstates, position, gn, hn)], dtstate)
                    state = np.append(state, q, 0)
                    # f(n)은 노드에 도달하는 비용과 노드에서 목표 상태까지의 비용의 합입니다.
                    fn = gn + hn                                        
             
                    q = np.array([(len(state) - 1, fn)], dtpriority)
                    priority = np.append(priority, q, 0)
                    # openstates의 노드가 목표 상태와 일치하는지 확인합니다.
                    if np.array_equal(openstates, goal):
                        print(' The 8 puzzle is solvable ! \n')
                        return state, len(priority)
         
                         
    return state, len(priority)
 
 
 
# 잘못 배치된 타일 휴리스틱을 사용한 8 퍼즐 평가 시작
def evaluvate_misplaced(puzzle, goal):
    steps = np.array([('up', [0, 1, 2], -3),('down', [6, 7, 8],  3),('left', [0, 3, 6], -1),('right', [2, 5, 8],  1)],
                 dtype =  [('move',  str, 1),('position', list),('head', int)])
 
    dtstate = [('puzzle',  list),('parent', int),('gn',  int),('hn',  int)]
 
    costg = coordinates(goal)
    # 부모, gn, hn을 초기화합니다. 여기서 hn은 misplaced_tiles 함수 호출 결과입니다.
    parent = -1
    gn = 0
    hn = misplaced_tiles(coordinates(puzzle), costg)
    state = np.array([(puzzle, parent, gn, hn)], dtstate)
 
    # 위치를 키로, f(n)을 값으로 하는 우선순위 큐를 사용합니다.
    dtpriority = [('position', int),('fn', int)]
 
    priority = np.array([(0, hn)], dtpriority)
    
    while 1:
        priority = np.sort(priority, kind='mergesort', order=['fn', 'position'])
        position, fn = priority[0]
        # 병합 정렬을 사용하여 우선순위 큐를 정렬합니다. 탐색할 첫 번째 요소를 선택합니다.
        priority = np.delete(priority, 0, 0)
        puzzle, parent, gn, hn = state[position]
        puzzle = np.array(puzzle)
        # 입력된 퍼즐에서 빈 칸(0)을 찾습니다.
        blank = int(np.where(puzzle == 0)[0][0])
        # g(n) 비용을 1 증가시킵니다.
        gn = gn + 1
        c = 1
        start_time = time.time()
        for s in steps:
            c = c + 1
            if blank not in s['position']:
                # 현재 상태를 복사하여 새로운 상태를 생성합니다.
                openstates = deepcopy(puzzle)
                openstates[blank], openstates[blank + s['head']] = openstates[blank + s['head']], openstates[blank]
                # 노드가 이전에 탐색되었는지 확인하기 위해 check 함수가 호출됩니다.
                if ~(np.all(list(state['puzzle']) == openstates, 1)).any():
                    end_time = time.time()
                    if (( end_time - start_time ) > 2):
                        print(" The 8 puzzle is unsolvable \n")
                        break
                    # Misplaced_tiles 함수를 호출하여 비용을 계산합니다.
                    hn = misplaced_tiles(coordinates(openstates), costg)
                    # 새로운 상태를 생성하여 리스트에 추가합니다.
                    q = np.array([(openstates, position, gn, hn)], dtstate)
                    state = np.append(state, q, 0)
                    # f(n)은 노드에 도달하는 비용과 노드에서 목표 상태까지의 비용의 합입니다.
                    fn = gn + hn                                        
                     
                    q = np.array([(len(state) - 1, fn)], dtpriority)
                    priority = np.append(priority, q, 0)
                    # openstates의 노드가 목표 상태와 일치하는지 확인합니다.
                    if np.array_equal(openstates, goal):
                        print(' The 8 puzzle is solvable \n')
                        return state, len(priority)
                         
    return state, len(priority)
 
 
# ----------  프로그램 시작 -----------------
 
# 초기 상태에 대한 사용자 입력
puzzle = []
print(" Input vals from 0-8 for start state ")
for i in range(0,9):
    x = int(input("enter vals :"))
    puzzle.append(x)
 
# 목표 상태에 대한 사용자 입력       
goal = []
print(" Input vals from 0-8 for goal state ")
for i in range(0,9):
    x = int(input("Enter vals :"))
    goal.append(x)
 
 
n = int(input("1. Manhattan distance \n2. Misplaced tiles"))
 
if(n ==1 ): 
    state, visited = evaluvate(puzzle, goal)
    bestpath = bestsolution(state)
    print(str(bestpath).replace('[', ' ').replace(']', ''))
    totalmoves = len(bestpath) - 1
    print('Steps to reach goal:',totalmoves)
    visit = len(state) - visited
    print('Total nodes visited: ',visit, "\n")
    print('Total generated:', len(state))
 
if(n == 2):
    state, visited = evaluvate_misplaced(puzzle, goal)
    bestpath = bestsolution(state)
    print(str(bestpath).replace('[', ' ').replace(']', ''))
    totalmoves = len(bestpath) - 1
    print('Steps to reach goal:',totalmoves)
    visit = len(state) - visited
    print('Total nodes visited: ',visit, "\n")
    print('Total generated:', len(state))
 
    print("Puzzle 상태:", puzzle)
    print("np.where 결과:", np.where(puzzle == 0))
    print("np.where[0] 결과:", np.where(puzzle == 0)[0])
 
if not is_solvable(puzzle):
    sys.exit("이 퍼즐은 해결할 수 없습니다.")
 
state, visited = evaluvate(puzzle, goal)
