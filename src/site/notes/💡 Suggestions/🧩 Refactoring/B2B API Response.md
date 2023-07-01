---
{"dg-publish":true,"permalink":"/suggestions/refactoring/b2-b-api-response/","noteIcon":"","created":"","updated":""}
---

# #refactoring 🧩 B2B API Response

# 🚧 Опис проблеми

Наразі дані і логіка респонсу змішані.

```php
readonly class CityCollection implements ApiResponseInterface  
{  
	/**  
	* @param DirectionCity[] $data  
	*/  
	public function __construct(  
		private array $data = [],  
		private bool $isProcessing = false,  
	) {  
	}
}
```

- не всюди потрібно і респонс і дані
- якщо зміниться структура респонсу, то не треба буде змінювати класи даних
---

# 🚀 Пропозиція

Пропоную відділити дані від респонсу:

```php
/**
 * @template T
 */
interface ApiResponseInterface extends JsonSerializable
{
    /**
     * @return  T
     */
    public function data(): JsonSerializable;
    public function status(): ResponseStatus;
    public function isProcessing(): bool;
    /* .... Other status checking methods */
    public function isNotAllowed(): bool;
}
class ApiResponse implements ApiResponseInterface
{
    public function __construct(
        protected readonly JsonSerializable $data,
        protected readonly ResponseStatus $status,
    ) {
    }
    public static function ok(JsonSerializable $data): static
    {
        return new static($data, ResponseStatus::OK);
    }
    /**
     * @return T
     */
    public function data(): JsonSerializable
    {
        return $this->data;
    }
    public function status(): ResponseStatus
    {
        return $this->status;
    }
    public function isProcessing(): bool
    {
        return $this->status === ResponseStatus::PROCESSING;
    }
}
/**
 * @implements ApiResponseInterface<BusTicketVoucher>
 */
class BusTicketVoucherApiResponse extends ApiResponse
{
    public function __construct(
        BusTicketVoucher $data,
        ResponseStatus $status,
    ) {
        parent::__construct($data, $status);
    }
}

```