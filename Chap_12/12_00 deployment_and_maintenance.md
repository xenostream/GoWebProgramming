# Deployment and Maintenance

현재까지 우리는 어떻게 프로그램을 개발하거나 소개했습니다. 프로그램 디버깅 및 테스트에는 개발의 마지막 10 %는 90 %의 시간을 필요로하는 것으로 알려져 있습니다. 따라서이 장에서는 마지막 10 %의 부분을 강조하여 신용 사용에 충분한 우수한 응용 프로그램이되도록 세부 사항을 고려해야합니다. 위의 10 %는 이러한 세부 사항을 가리 킵니다.

이 장에서는 4 절에 따라 이러한 세부 사항의 처리를 소개합니다. 제 1 절에서는 서버에서 프로그램이 생성하는 로그를 어떻게 기록하는지 소개합니다. 제 2 절에서는 오류가 발생했을 때 우리의 프로그램이 어떻게 처리되는지와 사용자의 액세스에 미치는 영향을 최대한 적게하려면 어떻게해야하는지 소개합니다. 세 번째 섹션은 Go 독립적 인 프로그램을 어떻게 배치하는지 소개합니다. 현재 Go 프로그램은 여전히 C처럼 daemon을 쓸 수 없습니다. 에서는 이러한 프로세스와 백엔드 프로그램을 어떻게 실행해야합니까? 제 4 절에서는 응용 프로그램 데이터의 백업과 복원을 소개합니다. 응용 프로그램이 깨진 상황에서 가능한 데이터 무결성을 보장합니다.
