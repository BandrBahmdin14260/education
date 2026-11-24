@startuml
autonumber
actor "Customer / Patient" as Customer
participant "Mobile / Web UI" as UI
participant "Order & Prescription Service" as Service
participant "OCR & Safety Engine" as Safety
actor "Pharmacist" as Pharmacist
database "Pharmacy Cloud DB" as DB

Customer -> UI : searchMedicine(query, filters)
activate UI
UI -> Service : getCatalogAndStock(query, filters)
activate Service
Service -> DB : queryMedicinesAndStock()
activate DB
DB --> Service : stockResults
deactivate DB
Service --> UI : catalogAndStock
deactivate Service
UI --> Customer : displayResults
deactivate UI

Customer -> UI : uploadPrescription(file)
activate UI
UI -> Service : submitPrescription(customerId, file)
activate Service
Service -> Safety : extractAndCheckSafety(customerId, file)
activate Safety
Safety -> DB : getAllergiesAndActivePrescriptions(customerId)
activate DB
DB --> Safety : allergiesAndPrescriptions
deactivate DB
Safety --> Service : ocrDataAndSafetyReport
deactivate Safety

Service -> Pharmacist : notifyForReview(ocrData, safetyAlertReport)
activate Pharmacist
Pharmacist --> Service : submitDecision(status, notes)
deactivate Pharmacist

Service -> DB : savePrescriptionAndAuditLog(decision, pharmacistId)
activate DB
DB --> Service : auditSaved
deactivate DB

alt Approved
    Service --> UI : prescriptionApproved
    UI --> Customer : promptDeliveryAndPayment
    Customer -> UI : confirmOrderAndPay(fulfillment, paymentMethod)
    UI -> Service : placeOrder(orderDetails)
    Service -> DB : deductStockAndCreateInvoice(orderDetails)
    activate DB
    DB --> Service : orderConfirmedAndInvoice
    deactivate DB
    Service --> UI : orderSuccess(invoice)
    UI --> Customer : showOrderAndInvoice
else Rejected / Clarification Needed
    Service --> UI : prescriptionRejected(reason)
    UI --> Customer : showRejectionNotice
end

deactivate Service
deactivate UI
@enduml
