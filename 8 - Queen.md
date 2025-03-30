# 8-Queen Problem Solver using Genetic Algorithm (No Crossover)

이 프로젝트는 8-Queen 문제를 유전자 알고리즘을 사용하여 해결하는 파이썬 프로그램입니다. 교차 연산(crossover)을 사용하지 않는 프로토타입 형태로 구현되었습니다.

## 8-Queen 문제란?
8-Queen 문제는 8x8 체스보드에서 8개의 퀸을 배치하는 문제로, 다음과 같은 조건을 만족해야 합니다:
- 각 퀸은 서로 공격할 수 없어야 합니다.
- 즉, 같은 행, 열, 또는 대각선에 두 개 이상의 퀸이 있어서는 안 됩니다.

## 유전자 알고리즘(Genetic Algorithm)이란?
유전자 알고리즘은 자연 선택의 원리를 모방하여 최적의 해를 찾는 탐색 기법입니다. 이 프로젝트에서는 교차 연산을 제외하고 다음과 같은 단계로 진행됩니다:
1. **초기 개체군 생성**: 랜덤으로 여러 해(개체)를 생성합니다.
2. **적합도 평가**: 각 개체의 적합도를 계산합니다.
3. **선택**: 적합도가 높은 개체를 선택합니다.
4. **돌연변이**: 선택된 개체에 무작위로 변화를 주어 새로운 개체를 생성합니다.
5. **반복**: 목표에 도달할 때까지 위 과정을 반복합니다.

## 구현 세부사항
- **개체 표현**: 8-Queen 문제에서는 하나의 개체를 8개의 숫자로 표현합니다. 리스트 `[a1, a2, ..., a8]`에서 `ai`는 i번째 열에 있는 퀸의 행 번호를 의미합니다.
- **적합도 함수**: 퀸들이 서로 공격하는 쌍의 수를 계산합니다. 공격하는 쌍이 적을수록 적합도가 높습니다. 적합도는 공격 쌍의 수에 음수를 취한 값으로 정의됩니다.
- **돌연변이**: 각 열에 대해 일정 확률로 퀸의 행 위치를 무작위로 변경합니다.
- **목표**: 공격하는 쌍의 수가 0이 되는 해를 찾는 것입니다.
- **교차 연산 제외**: 이 프로토타입에서는 교차 연산을 사용하지 않고, 돌연변이만으로 새로운 개체를 생성합니다.

## 코드 구조
- `EightQueensGA` 클래스: 유전자 알고리즘의 주요 로직을 포함합니다.
  - `initialize_population()`: 초기 개체군을 생성합니다.
  - `fitness()`: 개체의 적합도를 계산합니다.
  - `mutate()`: 돌연변이 연산을 수행합니다.
  - `run()`: 유전자 알고리즘을 실행하여 해를 찾습니다.
  - `print_board()`: 최종 해를 체스보드 형태로 출력합니다.

# 파이썬 코드 

import random
import copy

# 8-Queen 문제를 유전자 알고리즘으로 해결하는 클래스
class EightQueensGA:
    def __init__(self, population_size=100, mutation_rate=0.1, max_generations=1000):
        self.board_size = 8  # 체스보드 크기 (8x8)
        self.population_size = population_size  # 개체군 크기
        self.mutation_rate = mutation_rate  # 돌연변이 확률
        self.max_generations = max_generations  # 최대 세대 수
        self.population = self.initialize_population()  # 초기 개체군 생성

    # 초기 개체군 생성
    def initialize_population(self):
        population = []
        for _ in range(self.population_size):
            # 각 열에 대해 무작위로 행 번호를 선택 (0~7)
            individual = [random.randint(0, self.board_size - 1) for _ in range(self.board_size)]
            population.append(individual)
        return population

    # 적합도 계산: 공격하는 퀸 쌍의 수를 계산 (수가 적을수록 좋은 해)
    def fitness(self, individual):
        attacking_pairs = 0

        # 모든 퀸 쌍에 대해 공격 여부 확인
        for i in range(self.board_size):
            for j in range(i + 1, self.board_size):
                # 같은 행에 있는 경우
                if individual[i] == individual[j]:
                    attacking_pairs += 1
                # 대각선에 있는 경우
                if abs(individual[i] - individual[j]) == abs(i - j):
                    attacking_pairs += 1

        # 적합도는 공격 쌍의 수를 음수로 반환 (작을수록 좋음)
        return -attacking_pairs

    # 돌연변이 연산
    def mutate(self, individual):
        mutated = copy.deepcopy(individual)
        for i in range(self.board_size):
            if random.random() < self.mutation_rate:
                # 무작위로 행 위치 변경
                mutated[i] = random.randint(0, self.board_size - 1)
        return mutated

    # 유전자 알고리즘 실행
    def run(self):
        for generation in range(self.max_generations):
            # 개체군을 적합도 기준으로 정렬
            self.population.sort(key=self.fitness, reverse=True)

            # 가장 적합한 개체 확인
            best_individual = self.population[0]
            best_fitness = self.fitness(best_individual)

            # 목표 도달 (공격 쌍이 0) 시 종료
            if best_fitness == 0:
                print(f"Solution found in generation {generation}!")
                print("Best individual:", best_individual)
                print("Fitness:", best_fitness)
                return best_individual

            # 새로운 개체군 생성
            new_population = [best_individual]  # 엘리트 개체 보존
            while len(new_population) < self.population_size:
                # 상위 50%에서 무작위로 선택
                parent = random.choice(self.population[:self.population_size // 2])
                # 돌연변이 적용
                child = self.mutate(parent)
                new_population.append(child)

            self.population = new_population

        # 최대 세대까지 도달 시 가장 좋은 해 반환
        best_individual = self.population[0]
        print("No solution found within max generations.")
        print("Best individual:", best_individual)
        print("Fitness:", self.fitness(best_individual))
        return best_individual

    # 체스보드 출력
    def print_board(self, individual):
        for row in range(self.board_size):
            line = ""
            for col in range(self.board_size):
                if individual[col] == row:
                    line += " Q "
                else:
                    line += " . "
            print(line)
        print()
if __name__ == "__main__":
    # 유전자 알고리즘 객체 생성
    ga = EightQueensGA(population_size=100, mutation_rate=0.1, max_generations=1000)
    # 알고리즘 실행
    solution = ga.run()
    # 결과 출력
    print("\nFinal Board:")
    ga.print_board(solution)
