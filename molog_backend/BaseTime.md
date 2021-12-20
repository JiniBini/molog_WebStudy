# 생성/수정 시간 컬럼 자동화
방명록에 댓글을 등록할 때, 댓글이 달린 생성 시간과 수정 시간을 직접 입력하기에는 번거롭고 코드가 지저분해질 수 있습니다.
그래서 생성/수정 시간 컬럼을 Jpa Auditing을 이용해 자동화하여 구현했습니다.

## BaseTime Entity 생성
```
@MappedSuperclass
@EntityListeners(AuditingEntityListener::class)
abstract class BaseTime {
    @CreationTimestamp
    @Column(nullable = false, updatable = false)
    val createdAt: LocalDateTime? = null

    @CreationTimestamp
    @Column(nullable = false)
    val updatedAt: LocalDateTime? = null
}
```
생성일자를 나타내는 컬럼인 createAt은 수정되었을 때 날짜가 최신화되지 않게 하기 위해 @Column에 updatable = false를 추가해줍니다.

## Jpa Auditing 어노테이션 추가
Jpa Auditing 어노테이션들을 활성화할 수 있게 application 클래스에 어노테이션을 추가합니다.
```
@EnableJpaAuditing
@SpringBootApplication
class MologApplication

fun main(args: Array<String>) {
    runApplication<MologApplication>(*args)
}
```
## BaseTime 상속
생성/수정 시간 컬럼을 자동화하고자 하는 Entity 클래스가 BaseTime을 상속받도록 해줍니다.
```
@Entity
class GuestBook(user: User, comment: String): BaseTime() {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long? = null

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    @JsonIgnore
    var user: User = user

    @Column(nullable = false)
    var comment: String = comment
}
```
