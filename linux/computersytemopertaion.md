
# 컴퓨터 시스템 연산

cpu와 장치 제어기들은 공용 버스로 연결되며, 이 버스를 통하여 공유 메모리에 접근할수 있다. 각 장치 제어기는 특정장치를 관리한다. CPU와 장치 제어기는 메모리 사이클을 얻기 위해 경쟁하면서 병렬실행될 수 있다. 공유 메모리에 대한 질서 있는 접근을 보장하기 위해 메모리 접근을 동기화 시킨다.

컴퓨터가 구동을 시작하기 위해서는 실행시킬 초기 프로그램을 가지고 있어야 한다. 이 초기 프로그램 (부트스트랩 프로그램)은 매우 단순한 경향을 보인다. 전형적인 컴퓨터 내의 읽기 전용 메모리 ROM 나 EEPROM에 저장되고 펌웨어라고 알려져있다. 이것은 CPU 레지스터를 시작으로 장치 제어기, 메모리 내용 등을 포함한 시스템의 모든 측면을 초기화한다. 또한 부트스트랩 프로그램은 운영체제를 적재하는 방법 및 실행을 시작하는 방법을 알아야한다. 이러한 목적을 달성하기 위해 부트스트랩 프로그램은 운영체제 커널을 찾아 메모리에 적재해야한다.