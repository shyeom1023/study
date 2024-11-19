# AWS

## EKS 종료시 아래 내용이 멀까?

돈이 많이 나올까바 인스턴스를 중지하고 싶은데, 그냥 bastion 서버 종료하는건 상관없지만, eks node group을 삭제 할땐 이상한 경고가 발생

> 다음 인스턴스가 Auto Scaling 그룹에 연결됩니다.
>
> - i-00c6ebcc51b874fbd ([eks-demo-ng-eac99734-9a0f-71f6-02f4-ff4c9708d5ec](https://ap-northeast-2.console.aws.amazon.com/ec2/home?region=ap-northeast-2#AutoScalingGroupDetails:id=eks-demo-ng-eac99734-9a0f-71f6-02f4-ff4c9708d5ec))
> - i-0c63cb716f38e3762 ([eks-demo-ng-eac99734-9a0f-71f6-02f4-ff4c9708d5ec](https://ap-northeast-2.console.aws.amazon.com/ec2/home?region=ap-northeast-2#AutoScalingGroupDetails:id=eks-demo-ng-eac99734-9a0f-71f6-02f4-ff4c9708d5ec))
>
> 인스턴스를 중지하면 Amazon EC2 Auto Scaling이 인스턴스를 종료하고 자동으로 대체 인스턴스를 시작할 수 있습니다. 종료하면 EBS 루트 볼륨이 삭제될 수 있습니다. 종료 및 교체를 방지하려면 인스턴스를 중지하기 전에 Auto Scaling 그룹에서 연결 해제하세요.

![image-20241115162329805](assets/image-20241115162329805.png)

### chatGPT의 답변

Amazon EC2 Auto Scaling은 Auto Scaling 그룹에 속한 인스턴스가 중지되거나 종료될 경우 자동으로 대체 인스턴스를 시작합니다. 하지만, EBS 루트 볼륨이 삭제되지 않도록 하거나 인스턴스를 종료 및 교체하는 것을 방지하고 싶다면, 해당 인스턴스를 **Auto Scaling 그룹에서 연결 해제**해야 합니다.

### 인스턴스를 Auto Scaling 그룹에서 안전하게 연결 해제하는 방법

1. **AWS 관리 콘솔에서 Auto Scaling 그룹으로 이동**
   - [Auto Scaling 그룹 콘솔](https://ap-northeast-2.console.aws.amazon.com/ec2/autoscaling)로 이동하여 인스턴스가 속한 Auto Scaling 그룹을 선택합니다.
2. **인스턴스 연결 해제**
   - Auto Scaling 그룹 상세 페이지에서 **활성 인스턴스** 목록에서 인스턴스를 선택합니다.
   - **인스턴스 연결 해제** 옵션을 선택합니다.
   - 이 작업을 통해 해당 인스턴스는 Auto Scaling 그룹의 관리 대상에서 제외되며, 중지하거나 종료해도 Auto Scaling 그룹에서 자동으로 대체 인스턴스를 시작하지 않습니다.
3. **인스턴스 중지 또는 종료**
   - 이제 안전하게 인스턴스를 중지하거나 종료할 수 있습니다. EBS 루트 볼륨도 삭제되지 않으며, Auto Scaling 그룹에서도 새로운 인스턴스를 생성하지 않습니다.
4. **다시 Auto Scaling 그룹에 연결할 필요가 있을 때**
   - 필요에 따라 인스턴스를 다시 Auto Scaling 그룹에 연결하려면 **인스턴스 재연결** 옵션을 사용하면 됩니다.

이 절차를 통해 인스턴스의 중지 및 종료 시 자동 대체 생성과 EBS 볼륨 삭제를 방지할 수 있습니다.